---
layout: post
title: "Detecting when an Android app backgrounds in 2018"
date: 2019-04-03 14:00:00 +0200
navtab: blog
---

![](/assets/android-background-header.jpg)

For years, Android Applications lacked something iOS/UIKit had for free since the age of dawn. Detecting when the user left your app in a reliable and consistent manner. Spoiler alert: this has recently changed.

## Before 2018

It doesn't take long to land on a search result that points to this [StackOverflow Post](https://stackoverflow.com/questions/4414171/how-to-detect-when-an-android-app-goes-to-the-background-and-come-back-to-the-fo/), and its convoluted list of solutions (including [mine](https://stackoverflow.com/questions/4414171/how-to-detect-when-an-android-app-goes-to-the-background-and-come-back-to-the-fo/19920353#19920353)).

The fact that the accepted answer is partially wrong, or that none of the other solutions managed to get it right after all these years, is a testament of what should have been easy in 2008, really wasn't.

It is still possible to get a reliable method working with all these tools; notably the combination of `ComponentCallbacks2` and time tracking and/or keeping count of the numbers of activities that are “resumed/started”, etc. If you combine all that into a series of black boxes, you can possibly obtain a reliable method that will let you know when your app has been sent to the background and when it has returned.

Applications that need to lock or request a PIN after you leave (like [1Password](https://1password.com/)), are/were probably using some of these methods or combinations of them. And it works. I also used it in two different apps and didn't hear back from users complaining about it.

But it was horrible code. Still is. And more importantly, prone to error or false reports.

Deterministic code is code that given the same input, the output will be the same. On the other hand, non-deterministic code will behave the opposite way. You could argue that the above solutions were deterministic in a way, because you knew how the logic should behave given the different inputs. Unfortunately, as the number of variables increase and their sources is more and more hidden from view, it becomes very difficult to know who/what can fail and when.

All the above solutions were tied to different components, that knew little or nothing about others and had no clear rules for all possible scenarios. And even when we (the developers of the world) managed to get a working solution, the entire structure was weak and, in some cases, unpredictable.

Some of the old solutions that we used until now, were very non-deterministic, because they involved one or more of the following:

- **Time**: Some solutions used time to determine “how much time elapsed since…”, which was then used to decide what to do. Time is fine, but who knows the right value? Nobody does, except, perhaps Google, and they may get it wrong. The execution is deterministic (10 ms is alwas 10 ms), but what happens between now and 10 ms from now in my device, is something I don't entirely control. If the phone is slow, 10 ms may not be enough. We wouldn't know. Bad idea. (*For the record, I did use a solution involving time and, in fact, Google's own new solution involves time, but one has to be very careful about using this as the sole method*).
- **Reference Counts**: This one is tricky; in paper, it looks great. In practice, not so much. Long story short: it will produce false positives and negatives, due to the asynchronous nature of Android lifecycle events. Multiple activities getting started/resumed at the same time, may not end up having the results one think it has. Look at the original accepted answer and comments to learn more about it.
- **The Infamous** `onTrimMemory(final int level)` **from** `ComponentCallbacks2` **and its flag** `TRIM_MEMORY_UI_HIDDEN`: This one is fine for a lot of things, but it's also prone to false positives or worse, missing an event. Long story short again: when you run low on memory, the event may be silently skipped by Android and you'd never hear about your “ui hidden". Bad idea.

Ultimately, there are a few more things that one can do and the recommended approach was to use a combination of all these. It's the equivalent of checking for null in 20 different places, 20 different times, every time, in the hopes that “well, if this one doesn't catch it, the other will". Bad idea.

## 2018 - The Year Of The Lifecycle.

In Google I/O 2017, we heard about a new thing: Architecture Components. Google was finally acknowledging that there were so many new paradigms and ways to architect an Android app, that an official voice was needed to *guide the sheep to their right places*. And so Google came up with [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html), aka: a set of more-or-less independent libraries that can help you better architect and manage your Android application.

I will definitely **not** go into any detail about those, except for one tiny piece of it. Among the myriad of new content in Architecture Components, there is one that lives isolated from the others and doesn't really need them: [`Lifecycle Components`](https://developer.android.com/topic/libraries/architecture/lifecycle.html).

I want to take this chance to mention that, at first, I wasn't too happy about the solution; it felt (and still does) like a giant: “Ok, we can't fix this android lifecycle problem, so instead, let's patch the problem with this solution on top and be happy about it".

I believe this is **still true** but in practice, lifecycle components have demonstrated to be very useful when dealing with this problem, even if you don't even touch other components (Room, LiveData, ViewModels, RXJava, etc.). Remember, you **will have to live with Android lifecycle *things* whether you like it or not**, the question is: Do you want to handle the problem yourself or would you rather delegate it to a Google library, provided by them, that will do it for you? I hope you choose the later in this case.

Enough ranting…

**Objective**: Have a reliable way to determine when my Android App is in the background and when it returns.

**Gradle**: At minimum, you will need (latest version as of April 2018):


	dependencies {
	    // Lifecycles only (no ViewModel or LiveData)
	    implementation "android.arch.lifecycle:runtime:1.1.1"
	    annotationProcessor "android.arch.lifecycle:compiler:1.1.1"
	}
	
*(source: [https://developer.android.com/topic/libraries/architecture/adding-components.html](https://developer.android.com/topic/libraries/architecture/adding-components.html))*

**BUT IT DOES NOT WORK**: *this seems to crash as of April 2018 because the lifecycle Runtime doesn't have some dependency. Instead you will need to (at least until we hear back from Google) use this version instead*:

	implementation "android.arch.lifecycle:extensions:1.1.1"
	annotationProcessor "android.arch.lifecycle:compiler:1.1.1"

The side-effect is that you’re including more code than you need for the purpose of this sample; in practice it may not be an issue if you also rely on other architecture components (ViewData/ViewModels, for example).

**Code**: [The sample app](https://github.com/Gryzor/androidProcessLifecycleOwnerSample) is very simple doesn't consider all the use cases you will need in *your* app; keep that in mind before loading the copy-paste machine gun.

It all starts in `SampleApp`, which extends `Application` and overrides `onCreate()`:


	import android.app.Application
	import android.arch.lifecycle.ProcessLifecycleOwner
	
	class SampleApp : Application() {
	
	    private val lifecycleListener: SampleLifecycleListener by lazy {
	        SampleLifecycleListener()
	    }
	
	    override fun onCreate() {
	        super.onCreate()
	        setupLifecycleListener()
	    }
	
	    private fun setupLifecycleListener() {
	        ProcessLifecycleOwner.get().lifecycle
	                .addObserver(lifecycleListener)
	    }
	}
	

*What is all this?*

`SampleApp` is just an Android Application, declared in the manifest like:

	<application
	    android:name=".SampleApp"

All the *fun*, happens in the `setupLifecycleListener()` function. Essentially **one line of code**:

	ProcessLifecycleOwner.get().lifecycle
                .addObserver(lifecycleListener)

That's all. For real.

The mastermind behind all this is `ProcessLifecycleOwner` which belongs to the folks at Mountain View (a.k.a.: Google). I suggest you [read this by yourself from the official documentation](https://developer.android.com/reference/android/arch/lifecycle/ProcessLifecycleOwner.html), but since some people are lazy… let's recap really fast what it does (at the risk of copyright infringement, I will just paste the actual Javadoc from Google's code):

> *Class that provides lifecycle for the whole application process.*
> 
> *You can consider this LifecycleOwner as the composite of all of your Activities, except that `ON_CREATE` will be dispatched once and `ON_DESTROY` will never be dispatched. Other lifecycle events will be dispatched with following rules: ProcessLifecycleOwner will dispatch `ON_START`, `ON_RESUME` events, as a first activity moves through these events. `ON_PAUSE`, `ON_STOP`, events will be dispatched with a delay after a last activity passed through them. This delay is long enough to guarantee that ProcessLifecycleOwner won't send any events if activities are destroyed and recreated due to a configuration change.*
> 
> *It is useful for use cases where you would like to react on your app coming to the foreground or going to the background and you don’t need a milliseconds accuracy in receiving lifecycle events*.

In the same words with emphasis: **It is useful for use cases where you would like to react on your app coming to the foreground or going to the background and you don’t need a milliseconds accuracy in receiving lifecycle events.**

Therefore, you are guaranteed to receive **whole application lifecycle** events (with certain gotchas, like lack of millisecond accuracy). This takes care of us tracking the non-deterministic operations above:

- **Time**: Not needed anymore, it's handled by the component.
- **Reference Counts**: Tracked by the framework, we only get application events and so no need to keep numbers around.
-  **`OnTrimMemory/ComponentCallbacks2`**: Also no longer needed for this particular case, since we get specific callbacks, the UI HIDDEN flag and its unreliability under certain scenarios is gone.

The important bits in Google’s Javadocs are:

- You get one `ON_CREATE`
- You don't get `ON_DESTROY` at all.
- You get all the `START`, `RESUME`, `PAUSE`, `STOP` with a **delay** after a last activity passed through them.

This is the most important item. The component **will only dispatch the last pause/stop** (and start/resume equivalents). This is what makes it different from a **regular** lifecycle observer. From what I have been reading (the source code), the delay is 700ms.

	static final long TIMEOUT_MS = 700; //mls

It's also why it's called `ProcessLifecycleOwner` and with its name, also come the associated side-effect: If your app has **more than one process**, then you will need to keep track of each process in its own listener.

Speaking of the listener, the actual events are received via the supplied listener: *(how many times can I type listener here before it starts sounding weird)*

	class SampleLifecycleListener : LifecycleObserver {
	
	    @OnLifecycleEvent(Lifecycle.Event.ON_START)
	    fun onMoveToForeground() {
	        Log.d("SampleLifecycle", "Returning to foreground…")
	    }
	
	    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
	    fun onMoveToBackground() {
	        Log.d("SampleLifecycle", "Moving to background…")
	    }
	}

Method names are `anythingYouWantThemToBe()` but keep in mind you need the correct annotations. Since the sample app has only **one process**, then this is fine, we don't need to register more components per process to track them all.

## What does my App do with this information?

Here is where things end for me and start for you, the app developer.

What you do when you get a callback, is entirely up to you. In most cases, since this lifecycle listener is such a small and isolated component (which is a great thing), you will need to have some sort of way to publish this information to interested parties, but before you go and start adding Dagger, RXJava, OttoBus, and over 200 libraries, consider that all you may need is truly either a [Mixin](https://en.wikipedia.org/wiki/Mixin) class or some singleton somewhere where you can truly notify about the current app's status.

**Do not add** more responsibilities to the above listener, inject another component into it and set a boolean, for example:

	class SampleLifecycleListener : LifecycleObserver {
	
	    @Inject
	    var component: MyLifecycleInterestedComponent
	    
	    @OnLifecycleEvent(Lifecycle.Event.ON_START)
	    fun onMoveToForeground() {
	        component.appReturnedFromBackground = true
	    }
	
	    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
	    fun onMoveToBackground() {
	    }
	}

That way, your `MyLifecycleInterestedComponent` can expose/deal with the value and *other* components can then inject the same `MyLifecycleInterestedComponent` should they need the same data by doing the same:

	if (component.appReturnedFromBackground) { // do something }

Who sets the above flag as `false`?

It depends. Whoever needs to. Possibly whoever consumed the value, like the above if statement. If more components may be interested at the same time, then you know you will need to delegate that and so forth. Such is the nature of modern app development, delegation and separation of concerns.

That is all. If you made it here, the sample app is located on [GitHub, here](https://github.com/Gryzor/androidProcessLifecycleOwnerSample).

## GOTCHAS

This is a work in progress section. Keep in mind that not all use cases can be accomplished with the same solution. Things that usually caused problems in the past:

- Turning the screen OFF didn't trigger the background event: This is not true anymore. The lifecycle components will at the very least, pause and most likely **stop** the activity and as such, the `onMoveToBackground()` method will be called. I have personally tested this, but feel free to download the sample app and test it yourself.
- API Restrictions: this is still true, but if you’re developing for something below API 14, then I don't want to know about you. I am sorry you have to maintain such amount of legacy and insecure code around and that because of that, your users have to pay for your time developing workarounds. API 14 was released on October 19th, 2011. Consider the insane amount of progress both iOS and Android have gone through in the last 5 years…
- Low Memory may cause the event not to fire: I have been unable to test this so far, but I am highly confident that, unless your phone is left with 10 bytes of free memory to the point where the Linux Kernel and the Android Operating System cannot guarantee that a synchronous call in the main loop cannot be guaranteed to be called, then you should be spending your time on a deserted island drinking your beverage of choice instead of worrying about this. But test for it and if you have more information, please let me know and I will gladly add the results and, perhaps, try to find a solution.
- *Open for future things* :)

---
*July 2020: This story originally appeared on Medium under [ProAndroidDev](https://proandroiddev.com/) which I recommend you browse often; for personal reasons I've chosen to leave medium behind, so I've exported my data and reproduced the Markdown using the tool [described in this post](https://medium.com/@macropus/export-your-medium-posts-to-markdown-b5ccc8cb0050) hosted at, you guessed, Medium. The irony.* 