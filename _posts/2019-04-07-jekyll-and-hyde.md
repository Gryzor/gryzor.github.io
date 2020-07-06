---
layout: post
title:  "Dr. Jekyll and Mr. Tumblr"
date:   2019-04-07 18:20:44 -0500
categories:
navtab: blog
---

Yesterday morning [this tweet from Mark Sands][rando] showed up in my mentions:

> **@jaredsinclair** Looks like Tumblr deleted your blog. Any hope for reviving it?

Up until yesterday, my blog was hosted via Tumblr with a CNAME redirect from `blog.jaredsinclair.com` to the appropriate Tumblr domain. I set everything up seven or eight years ago, and it had worked fine all that time. I tried visiting the site and, like Mark said, the blog was gone. I got the Tumblr equivalent of a 404 instead of my blog. Next I tried logging into my Tumblr account to figure out what was wrong. Here's what I saw:

<p class="inline-image"><a href="{{ site.url }}/assets/img/hyde-tmblr-account-terminated.jpeg"><img src="{{ site.url }}/assets/img/hyde-tmblr-account-terminated.jpeg"/></a></p>

_W. T. Fuck._

I thought perhaps I'd missed a warning from Tumblr, but I had no recollection of receiving anything. I double and triple checked all my email accounts. Nada. There wasn't anything from Tumblr since one day in March when I used a magic link to sign into my account rather than username and password. Nothing archived, nothing in spam. Without warning Tumblr had terminated my account for no discernable reason. I tried logging in from a desktop browser to be sure there wasn't some mobile-rendering goof. Same thing. _Your account has been terminated_ in classic Tumblr chunky white.

### Wait, back up.

Ever since I slung screwdrivers around repairing Macs at a [third-party Mac store][ma], I've lived in mortal fear of losing data. Practically every day we had to tell somebody _your baby pictures, your wedding pictures, your dissertation: all gone_. There's only so many times you can watch folks sob over the consequences of a spilled cup of coffee before you're sobbing with them. For the most part, my backup strategy has been comfortably paranoid:

- All photos and videos are backed up to my iCloud Photo Library. This entire library is synced to the family computer. The family computer is synced to Backblaze. Google Photos adds a redundant backup of everything, as well, using the iOS app. There's also a good chunk of our most important photos and videos in the Dropbox accounts of my family members. Last but not least, we print a book of photos every year so there's a hard copy that'll survive once I die and there's no one else in the family willing to be as paranoid as I am.

- Every paper document that comes through our house that has even a scant chance of being useful later gets digitized with [Scanner Pro][scanner] and Dropbox-ed. Scanner Pro has a cool feature where you can have every scan automatically synced to a predetermined Dropbox folder. At the start of each year, I update this to point to an `Inbox 20--` folder for that year. If I need to move something out of there to a more appropriate folder, I do, but otherwise I just let things accumulate. _The perfect is the enemy of the good_ and all that.

- Both my and my wife's Dropbox accounts are fully synced to the family computer, and thus also to Backblaze along with the rest of the stuff on it.

- Github: use it! I use it religiously, even for the small stuff, like the secret Gist I use to track my `.bash_profile`. I enable [branch protections][branch] on everything, too, just in case.

- An external hard drive or two, however outdated, are kept in our safe deposit box. So even if Apple, Dropbox, Google, GitHub, and everyone else all decide to terminate my accounts in one day, hopefully there will still be _something_ available on those old drives that I can salvage.

Yet my blog was the one corner of my digital life where I had gotten lazy. I didn't have a backup of my posts anywhere, only scattered draft versions in a Dropbox folder. The canonical versions of all my blog posts were whatever Tumblr had saved for my account. The Tumblr versions had lots of small edits and corrections that weren't saved anywhere, even if there was a draft copy in Dropbox. Now suddenly, everything that Tumblr had was gone.

### Moving to Jekyll

I used the spartan contact form to ask Tumblr why my account was terminated. But rather than wait for a response that might never come, I decided it was past time to bring my blogging setup in line with the rest of my backup paranoia.

I decided to move everything over to [Jekyll][jek], backed by a comprehensive GitHub repository. I've had a [Media Temple][mt] account for years and have been really happy with them. I knew once I figured out how to actually _use_ Jekyll in a comfortable way, my Media Temple Grid Service would be able to host the static files easily. Why Jekyll? The short answer is that it's used by GitHub Pages. In a sea of alternatives, I'm content to follow the smart folks at GitHub wherever they go.

If you're reasonably comfortable with basic web programming, and know enough about the shell to get yourself in trouble, using Jekyll isn't so bad. I got a proof-of-concept site up and running pretty quickly. Since Media Temple is also the domain registrar for `jaredsinclair.com`, it was easy to replace the Tumblr CNAME record for `blog.jaredsinclair.com` with an A record pointing to the same IP address as `jaredsinclair.com` (this was a necessary part of ensuring that old blog post links resolved to their new Jekyll permalink). Within hours, anyone that wanted to visit this blog was able to see _something_ here. The real problem was recovering all my old posts.

### Recovering all my old posts

Felix Lapalme [recommended][felix] a [command-line tool][archive-tool] that downloads an entire site's content from the Internet Wayback Machine. Before anything else could go wrong, I immediately ran that tool, which worked as advertised (isn't it great when things work like they say on the tin?). This archive was missing a lot of posts, particularly my more recent stuff, but something was better than nothing. I was pretty worn out from getting Jekyll set up so I went to bed, putting off figuring out how to transform this backup into something formatted for Jekyll.

When I woke up this morning, I had a pleasant email in my inbox. It wasn't from Tumblr, natch. It was from [Ben Ubois][ben], the founder of [Feedbin][feedbin], the RSS aggregator:

> Hey Jared,
>
> I saw on Twitter about Tumblr closing your account. That sounds lame!
>
> Feedbin has posts from your blog going back to 2012. I’ve attached all 234 of them as JSON.
>
> The structure looks like:
>
> { title: "Title", url: ...
>
> Hope it helps!

Thanks to Ben, I now had everything I needed. Unlike the Internet Archive backup, in which each post would need to be heavily transformed to unsleeve the post content from all the page chrome that got captured with each crawl, the RSS backup was already free of such chrome. Better still, the JSON data structure would make it possible to automate the capture of the critical metadata for each page, namely the title and published date.

Using the Swift Package Manager, I made a [quick n’ dirty utility][dirty] that transforms a JSON-encoded file of Feedbin posts into a directory of HTML files formatted for Jekyll. After running this utility, all I had to do to republish all my old content (in correct chronological order, too) was to copy those HTML files into the `_posts` directory in my Jekyll project, run `jekyll build`, and upload them to the right directory on my Media Temple server.

### Fixing broken links

The one truly unfortunate downside of moving to Jekyll is that all existing links to my Tumblr-hosted blog are now defunct. Tumblr blog post URL paths take one of the following forms:

```
/post/123456789
/post/123456789/title-slug-for-post
/post/123456789/title-slug-for-post/index.html
```

Whereas Jekyll links use a date-based path:

```
/2019/04/07/title-slug-for-post.html
```

At least that's the Jekyll default. I like this default and have decided to keep it and fix broken Tumblr links on a case-by-case basis. For the posts that I care about most (the ones that at one time or another got a lot of traffic, like [this one][unread] or [this one][design]), I've updated the `.htaccess` file on my Media Temple server with redirects like this:

```
RedirectMatch 301 /post/97655887470.*$ /2014/09/16/good-design-is-about-process-not.html
```

I don't know if I'll do it this way forever (there might be a better way that Jekyll supports), but this was effective and took only a few minutes.

For all the posts that I haven't redirected, I've updated the [404][fof] with a blurb about what happened to my blog this weekend, with a link to my [archive page][archive-page].

### Looking ahead

To make day-to-day life easier going forward, I added a script to my Jekyll project that uses [rsync][r] to upload the `_sites` directory to my Media Temple server, authenticated with ssh. From the root directory of my Jekyll repo, I can just run `publish.sh` to rebuild and upload everything. I'm continually impressed by how efficient rsync is. Small changes to the site, like correcting a typo in some markup, are published in seconds.

### Update: Tumblr replies

By the time I finished writing this post, I received a reply from Tumblr:

> Hello,
>
> We've restored your account.
>
> Thank you for bringing this problem to our attention. We're sorry that it occurred, and we'll do our best to make sure that it doesn't happen again.
>
> You should now be able to log in just fine with your email address and password.
>
> Please let me know if there's anything else I can help you with!
>
> Drew
>
> Community Manager

Too little, too late, Drew. I hope Tumblr understands that I simply cannot trust them anymore. I'm grateful that they responded to my contact request and that my account has been reopened, but it's unacceptable that a years-old account still in good standing can be terminated without any advance warning or preventative recourse. This weekend's debacle is a textbook case for why folks should own their own data. It's also a good reminder that no single service provider can be wholly trusted.

**If something isn't backed up in more than one place, it's not backed up at all.**

[rando]: https://twitter.com/marksands/status/1114597508037083137
[ma]: http://web.archive.org/web/20060111081706/http://www.macauthority.com:80/
[scanner]: https://readdle.com/scannerpro
[branch]: https://jaredsinclair.com/2019/03/24/think-twice-before-downgrading-t.html
[archive-tool]: https://github.com/hartator/wayback-machine-downloader
[jek]: https://jekyllrb.com
[mt]: https://mediatemple.net
[felix]: https://twitter.com/lap_felix/status/1114600403868557314
[ben]: https://twitter.com/bsaid
[feedbin]: https://feedbin.com
[dirty]: https://gist.github.com/jaredsinclair/7ea54d4e3e75e6394f72a53b5ed548df
[r]: https://rsync.samba.org

[fof]: {{ site.url }}/404.html

[archive-page]: {{ site.url }}/archive.html

[unread]: {{ site.url }}/2014/07/28/a-candid-look-at-unreads-first-y.html

[design]: {{ site.url }}/2014/09/16/good-design-is-about-process-not.html
