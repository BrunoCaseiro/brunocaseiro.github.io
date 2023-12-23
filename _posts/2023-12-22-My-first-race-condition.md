---
layout: single
title:  "My first race condition"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

## Introduction

Welcome to my first ever blog post. You are witnessing history. This is about the time I found a race condition during a penetration test. Many details and specificities will have to be left out due to confidential nature of issues like these - and this will probably be a constant throughout my blog. With this being said... lights out and away we go! ~~<sup>Verstappen leads heading into the first corner</sup>~~

<p align="center"> <img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/90450394-e1f0-41f7-b7bd-739ebf842860"> </p>


## What is a race condition?

This might seem like a hard concept to **understand**, but it's not **that** hard when it finally clicks. Exploiting it is a whole other beast, but during one of my teaching sessions about introductory pentesting, I went on a tangent (the students pulled it out of me!) and managed to explain this to people with minimal computer science knowledge. This is best understood with a concrete example, so I'll explain as we go. A good definition however, would be something like the following.

> _A race condition occurs when two or more threads can access shared data and they try to change it at the same time._

Think about two threads running the same program and working with the same variables - the order in which the operations happen will lead to very weird behaviors!

Coincidentally, I found this race condition the day after I read [James Kettle's research article on race conditions](https://portswigger.net/research/smashing-the-state-machine). It's a great read, I cannot recommend it enough. A video to his Defcon talk on the same topic is included.


## Application logic

Picture an application that stores an integer value X. You can set a new value Y, but there are a few things to note:
  - Let's assume there is a default starting value X, for simplicity's sake
  - If you choose to set a higher value Y, then your value will be kept on hold during a time period before being set
  - If you chose to set a lower value Y, your value is immediately set
  - The application continuously tracks the **ACTIVE** value and the **PENDING** value (the latter can be _null_)

After observing this behaviour, I assumed a simplified version of the code would be close to this:

```
def function(int y):
  if (y > x):
    wait_set(y)
  else:
    set(y)
```

We are still going over how this works when used correctly. So two things can happen here, depending on the the conditional statement:
  - If Y is larger than X --> ``ACTIVE = x`` and ``PENDING = y``
  - If Y is **not** larger than X --> ``ACTIVE = y`` and ``PENDING = null`` (this also applies if the variables are equal, but that won't matter)

Pretty simple, right? What can possibly go wrong?

## The problem

Suppose we launch two threads of the program at the same time and the variables are shared. So if thread 1 sets a value for Y, it is also set for T2. Obviously the odds of this happening are basically null, it requires super precise timing, right? Right....?

Not quite, we will go over how James Kettle made this so much easier to exploit later. But first, let's go back to our little problem - what if we could fail the if statement (Y value lower than X) and then immediately change Y to a value higher than X mid execution, so when the line ``set(y)`` is ran, **a value higher than X will become active without a cooldown**?

Let's look at the diagram below, hopefully my handwriting isn't bad enough to make you click away <sup>pls stay</sup>. On the left we have thread 1, on the right thread 2. The initial values are shown above the code, which is there for quick reference. The circled numbers are the order in which the operations happen. The race condition could also happen if the operations happen in a different (similar) order, this is just one possible example.

<p align="center"> <img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/f2d34a21-86bb-4eb1-9f9e-b93cdff99a32"></p>

⚪0) Starting with ``ACTIVE = 50`` and ``PENDING = null``

🔴1) T1 sets ``y = 49``

🔴2) T1 fails the if condition since ``49 > 50 ? False``

🟢3) T2 sets ``y = 51``

🔴4) T1 calls ``set(y)``, which translates to ``set(51)`` since T2 just changed it --> ``ACTIVE = 51`` and ``PENDING = null``

🟢5) T2 passes the if condition since ``51 > 50 ? True``

🟢6) T2 ``calls wait_set(51)`` -->  ``ACTIVE = 51`` (same as the latest value) and ``PENDING = 51``

   
This was the idea I had in mind. I do not know exactly how the code is written, but that last step made me itch. Calling ``wait_set()`` with a value equal to ``ACTIVE``? Maybe there is some pre-condition in case that happens?

I refreshed the page and ``ACTIVE`` was set to 51! I did it! Later I refreshed the page again. ``ACTIVE`` was set to 49. What? I refreshed a couple of more times and the page would just show one of the values, no pattern or consistent behaviour. I tried setting a new value legitimately and I got an error. I analyzed the requests with Burp Suite and noticed something extra weird. ``ACTIVE = 51``, ``ACTIVE = 49`` and ``PENDING = null``. I actually set **two** active values! I did try the attack a couple more times but this was very time consuming. When succesful, this would always be the result.


## Timing is the key

