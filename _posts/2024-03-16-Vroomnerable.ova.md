---
layout: single
title:  "Vroomnerable.ova"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

## Introduction

I started this blog a few months ago with [this post](https://brunocaseiro.github.io/My-first-race-condition/) about a race condition I found. I refuse to contribute to the internet's blog graveyard, so here we are.

Anyway, this led to a similar blog post, this time on [Blip's blog](https://www.blip.pt/blog/posts/test-drive-the-challenges-of-race-conditions-in-security-testing/) and eventually a conference talk at [ØxＯＰＯＳɆＣ](https://www.meetup.com/0xoposec/), which at the time of writing has not happened yet. Our team's manager had the cool idea of building a simple demo web application I could use at the talk (thanks [João](https://pt.linkedin.com/in/joao-morais-84aa4936)! :D).

I took the web app and turned it into a small challenge based on my initial finding. Then I decided to go a step further and build an entire virtual machine around it.


## The challenge


I'll try not to spoil it too much (**yet**, walkthrough below!), but it is a pretty simple system from an architectural point of view. There's a web application and an SSH server running. After hacking into the **vroomnerable** user, you get to escalate your privileges to **root** by exploiting another race condition - this second part is where it can get a bit tricky.

I confess the machine is a bit CTF-y, it's not supposed to simulate a real situation or environment. They are just challenges presented in a straight forward way, with no rabbit holes and no need to read between the lines.

Here is the link to download the .ova file. I promise it's not malware :)
- <https://drive.google.com/file/d/1MOE3WM5P_lae8zgQ5kTqCXC2QeF_sNbM/view?usp=drive_link>




It's the first time I'm building a vulnerable VM, so I'm not expecting it to be perfect. But come on, give it a go! Write a walkthrough or give me your feedback. Don't forget to ping me!






# SPOILERS AHEAD

## Walkthrough



Thanks for reading

Bruno Caseiro
