---
layout: post
title: "Android ConstraintLayout Chains & RTL Workarounds (Updated)"
date: 2017-12-05 14:00:00 +0200
navtab: blog
---

**NOTE**: Google has fixed (or claims to, for I haven't had the time to check it yet) this in the latest Beta 4. At this time, I don't know if the version will be 1.0.4 or 1.0.3 or X.X.X, but if you implement the workaround described below, leave a `//TODO` to check back when the fix is released “sometime during 2018".

In order to form a Chain in a Constraint Layout, the framework rules state that:

1. The **leftmost** item's **left** side, must be constrained to the **parent’s left** and the **rightmost** item’s **right** must be constrained to the parent’s **right**.
2.  All the elements of the chain must do a circular reference between themselves. So a chain between two elements looks like:

| ← **A** ← → **B** →|

Where the "box" is the parent.

3. Finally the type of chain **must be specified in the first element of the chain** (left for horizontal, top for vertical). This has nothing to do with LTR or RTL, the engine counts from left to right; the “first" element is the left-most one, or **A** in the diagram above. This element is called the **head of the chain**. And if you read the [Google documentation](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html#Chains), it's the left-most item of the chain.

You will obviously want to use `start/end` instead of `left/right` to fully support RightToLeft languages.

The problem is RTL languages `start/end` flip everything automatically, your margins, constraints, etc., are all going to do the right (no pun intended!) thing, except the “first item of the chain" is still the `left-most` item, not the “start item".

To better illustrate the problem, imagine this layout:

	<?xml version="1.0" encoding="utf-8"?>
	<android.support.constraint.ConstraintLayout 
	  xmlns:android="http://schemas.android.com/apk/res/android"
	  xmlns:app="http://schemas.android.com/apk/res-auto"
	  android:layout_width="match_parent"
	  android:layout_height="match_parent">
	
	  <TextView
	    android:id="@+id/text_one"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:text="Text One"
	    app:layout_constraintEnd_toStartOf="@+id/text_two"
	    app:layout_constraintHorizontal_chainStyle="packed"
	    app:layout_constraintStart_toStartOf="parent"
	    app:layout_constraintTop_toTopOf="parent" />
	
	  <TextView
	    android:id="@+id/text_two"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:text="Text Two"
	    app:layout_constraintEnd_toEndOf="parent"
	    app:layout_constraintStart_toEndOf="@id/text_one"
	    app:layout_constraintTop_toTopOf="parent" />
	</android.support.constraint.ConstraintLayout>

A simple `ConstraintLayout` that has two `TextView` that are forming a Horizontal Chain, packed.

In a regular LTR language, this looks like this:

![](/assets/constrainlayout-rtl-image1.png)

But when you switch to RTL…

![](/assets/constrainlayout-rtl-image2.png)

### What happened here?

`tl;dr`: The Chain Broke.

Because TextOne is no longer the “left most item” (aka: the Head) of the chain, the chain requirement that the “first / left item" must define the chain style, is now broken, because the chain style is now defined in TextOne, and TextTwo is the left most item. This makes TextTwo the Head of the chain in RTL.

**Enough theory, what’s the fix?**

Until I hear more about Google (see Bug report posted [here](https://issuetracker.google.com/issues/70181930)), the solution I found to work and have no “side effects so far" is to **duplicate the chain style declaration in both the first/left and last/right element of the chain**.

It looks like this:

	<?xml version="1.0" encoding="utf-8"?>
	<android.support.constraint.ConstraintLayout 
	  xmlns:android="http://schemas.android.com/apk/res/android"
	  xmlns:app="http://schemas.android.com/apk/res-auto"
	  android:layout_width="match_parent"
	  android:layout_height="match_parent">
	
	  <TextView
	    android:id="@+id/text_one"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:text="Text One"
	    app:layout_constraintEnd_toStartOf="@+id/text_two"
	    app:layout_constraintHorizontal_chainStyle="packed"
	    app:layout_constraintStart_toStartOf="parent"
	    app:layout_constraintTop_toTopOf="parent" />
	
	  <TextView
	    android:id="@+id/text_two"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:text="Text Two"
	    app:layout_constraintEnd_toEndOf="parent"
	    app:layout_constraintHorizontal_chainStyle="packed"
	    app:layout_constraintStart_toEndOf="@id/text_one"
	    app:layout_constraintTop_toTopOf="parent" />
	</android.support.constraint.ConstraintLayout>

The only addition was `app:layout_constraintHorizontal_chainStyle="packed”` to the second text view (the one that will become **head of the chain** in RTL).

---
*July 2020: This story originally appeared on Medium; for personal reasons I've chosen to leave Medium behind and exported my data using the tool [described in this post](https://medium.com/@macropus/export-your-medium-posts-to-markdown-b5ccc8cb0050) hosted at, you guessed, Medium. The irony.* 