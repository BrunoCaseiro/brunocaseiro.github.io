---
layout: single
title:  "My first race condition"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

## Introduction

Welcome to my first ever blog post. You are witnessing history. This is about the time I found a race condition during a penetration test. Many details and specificities will have to be left out due to the confidential nature of issues like these - and this will probably be a constant throughout my blog. With this being said... lights out and away we go! ~~<sup>Verstappen leads the way heading into the first corner</sup>~~

<p align="center"> <img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/90450394-e1f0-41f7-b7bd-739ebf842860"> </p>


## What is a race condition?

This might seem like a hard concept to **understand**, but it's not **that** hard when it finally clicks. Yes, exploiting it is a whole other beast. I went on a tangent during one of my teaching sessions about introductory pentesting (the students pulled it out of me!) and managed to successfully explain this to people with freshman-level computer science knowledge. This is best understood with a concrete example, so I'll explain as we go. Just so we have a starting point, a good definition would be something like the following.

> _A race condition occurs when two or more threads can access shared data and they try to change it at the same time._

Think about two threads running the same program and working with the same variables - the order in which the operations happen will lead to very weird behaviors!

Coincidentally, I found this race condition the day after I read [James Kettle's research article on race conditions](https://portswigger.net/research/smashing-the-state-machine). It's a great read, I cannot recommend it enough. A video to his Defcon talk on the same topic is included in his post.


## Application logic

Picture an application that stores an integer value X. You can set a new value Y, but there are a few things to consider:
  - Let's assume there is a default starting value X, for simplicity's sake
  - If you choose to set a higher value Y, then your value will be kept on hold during a time period before being set
  - If you choose to set a lower value Y, your value is immediately set
  - The application continuously tracks the **ACTIVE** value and the **PENDING** value (the latter can be _null_)

After observing this behavior, I assumed a simplified version of the code would be close to this:

```
def function(int y):
  if (y > x):
    wait_set(y)
  else:
    set(y)
```

We are still going over how this works when used **correctly**, so two things can happen here depending on the conditional statement:
  - If Y is larger than X --> ``ACTIVE = x`` and ``PENDING = y``
  - If Y is **not** larger than X --> ``ACTIVE = y`` and ``PENDING = null`` (this also applies if the variables are equal, but that won't matter too much)

Pretty simple, what can possibly go wrong?

## The problem

Suppose we launch two threads of the program at the same time and the variables are shared. So if thread 1 sets a value for Y, it is also set for thread 2. Obviously the odds of something weird happening are basically null, it requires super precise timing, right? Right....?

Not quite, we will go over how James Kettle made this so much easier to exploit later. But first, let's go back to our little problem - what if we could fail the if statement (Y value lower than X) and then immediately change Y to a value higher than X mid execution, so when the line ``set(y)`` is ran, **a value higher than X will become active without a cooldown**?

Let's look at the diagram below, hopefully my handwriting isn't bad enough to make you click away <sup>pls stay</sup>. On the left we have thread 1, on the right thread 2. The initial values are shown above the code, which is there for quick reference. The circled numbers are the order in which the operations happen.
(Update: the diagram is now very pretty).


<p align="center"><img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/66176572-f97c-48e2-a1d7-b1ddeddf0b28"></p>

⚪0) Starting with ``ACTIVE = 50`` and ``PENDING = null``

🔴1) T1 sets ``y = 49``

🔴2) T1 fails the if condition since ``49 > 50 ? False``

🟢3) T2 sets ``y = 51``

🔴4) T1 calls ``set(y)``, which translates to ``set(51)`` since T2 just set the Y variable to 51 --> ``ACTIVE = 51`` and ``PENDING = null``

🟢5) T2 passes the if condition since ``51 > 50 ? True``

🟢6) T2 ``calls wait_set(51)`` -->  ``ACTIVE = 51`` (same as the latest value) and ``PENDING = 51``

   
This was the idea I had in mind. I do not know exactly how the code is written, but that last step made me itch. Calling ``wait_set()`` with a value equal to ``ACTIVE``? Maybe there are some pre-conditions in case weird things like these happen?

I was using Burp Suite's repeater tab to send these requests. Simply put both tabs in a folder and selecting **Send group in parallel (single-packet attack)**. The example below is sending the 3 requests in the _limit-overrun_ folder using this technique.

<p align="center"><img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/bad40add-60f1-4d16-8059-4b3674a9a36f"></p>

It took several attempts of sending 2 requests in a single packet (one with a higher value than ``ACTIVE``, other with a lower value) to make this actually work. The server would act as if only one of the values was sent. No pattern or consistent behavior was noted, but one of the values would indeed be set. It would either overwrite ``ACTIVE`` (lower value) or set a new ``PENDING`` (higher value), depending on which one the server "chose". To not reuse the same values, I would increment the higher value and decrement the lower value after every attempt - for example, I'd send 51 and 49, then 52 and 48, 53 and 47 and so on.

After many, many attempts, I finally got a hit. But not something I was expecting. The response size suddenly shot up. I took a more careful look and I was amazed at what I was seeing. Two ``ACTIVE`` variables. One set to the higher value and another one set to the lower one. Even if I sent a request that simply fetched the current values, the response would be something like this:

```
ACTIVE = 51
ACTIVE = 49
PENDING = null
```

What just happened? I actually have two active values?

I tried setting a new value but the server response was a **500 Internal Server Error**. I tried performing some operations that required these values and once again... **500 Internal Server Error**. I reset the whole process, repeated the attack (which was very time consuming) and ended up in the same place. 

I would have loved to have some more time to explore this scenario. Perhaps I did not have enough luck during all these attempts and the order of the operations wasn't quite what I needed, but I feel like I got pretty close.

If I had managed to go a bit further, the impact of this vulnerability would likely be high (due to the context of the application), but *_unfortunately_ it didn't go past a "self denial of service".

*Food for thought: If you are a penetration tester, are you happy you found a cool vulnereability or sad that the application is broken? 😄


## Timing is the key

A race condition is so hard to exploit because there are a bunch of timing variables involved, and you have to be lucky enough to have each single line executed in a very specific order. Taken from James Kettle:

<p align="center"><img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/53e4fbb5-80ce-41d5-a39f-e1bbe70699d5"></p>

Even if you code your requests to be sent at the exact same time, there's your network latency, the jitter (time between transmission and reception of data) and the server's latency.

After some iterations, James' final solution is to send several requests in a single TCP packet. Which means the jitter and latencies involved will all be the same. There's still some trial and error due to the processing times on the server, but the effectiveness of this attack goes up exponentially.

<p align="center"><img src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/aca316bd-9347-47ff-9f29-ad356c44fb47"></p>

I did try the other techniques, such as using Burp Suite's intruder and last-byte sync, but the only time I got interesting results was when I sent the requests in a single TCP packet.

This is an extremely quick summary of James' entire research, so there is a lot of details missing here. I once again recommend that you take a look at his article, it really is great research!


## Conclusion

Hopefully you enjoyed my first blog post, I'll try to keep an eye out for other possibly interesting stories to share in order to keep this website alive. We already have enough dead blogs on the internet 😅.

Feel free to read more [about me](https://brunocaseiro.github.io/aboutme.html) or get in touch via [LinkedIn](https://linkedin.com/in/brunocaseiro)!

Thanks for reading

Bruno Caseiro
