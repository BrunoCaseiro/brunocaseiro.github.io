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

### Setting up
If you're using VirtualBox, importing the machine should be pretty straight forward after downloading the .ova file. Simple press **File --> Import appliance** and select the downloaded file. It should come pre-configured (I recommend setting your machine and the vulnerable machine to "NAT network", that never results in any network problems for me).

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/3d4bacea-3c6a-4c1b-8b6a-0bedd0582ae1)


You boot both machines (your Kali and vroomnerable) and if everything goes according to plan, they should be on the same network. Time to find out the victim's IP address.
The attacker's ip is **10.0.2.4/24**, so vroomnerable should be somewhere in the **10.0.2.0/24** network. The below command will scan every IP in that range.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/74f0e7e5-61c3-4b54-9ad3-b2868b1632f7)

To be sure which of the results is the target machine, you can compare the MAC addresses found to the VMs MAC address, which can be found in the network settings.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/0e9319c2-4b3c-4be3-aa8f-bf3cf549af21)


### Enumeration
This is a pretty basic nmap scan, but we don't need more than this. There's an SSH server and a web app running on port 8000.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/56ea8c7f-7f1a-42c5-a1ba-9c045199febd)

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/55f3ff13-0eb1-42e0-9202-2e4c05553169)

This seems like an impossible challenge at first, since you cannot set a higher active value, only lower ones. After performing all your enumeration steps, you should find a hint. Yes, it's a shamless plug to my blog (again 🙄).

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/6776e0bc-e5e9-40b7-bc27-57e671460d60)


### User flag
The exploit for this first challenge is exactly what's in the initial blog post, so I won't go deep into that. This time, it only took me 6 tries - starting at the values 49/150, down to 44/150.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/db53e99f-da69-45b5-8147-a333302ffce9)

These credentials work for SSH, so here is the user flag.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/4ae6ba9d-d9f2-48f6-bf70-b38b9950de43)

The source code is in the home folder. This is artifically vulnerable due to the **log_operation()** function. **wait_set()** sets a PENDING value, while **set()** sets an ACTIVE value.
When sending a lower value, the **else** branch is taken and the artifical delay starts. That's the race condition window and when the second value (150) should change the y variable before first thread completes.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/49071ab0-bb56-4aa0-ad09-8646af1333d3)

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/de14ede0-5f9c-48e5-8d99-3b4ca2102bf8)


### Root flag
This is where it can get tricky. At first, this is the only thing we have to escalate our privileges.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/6ab8d385-be0a-41bf-8447-01854371cb89)

Some files are being deleted and some files are being executed. The numbers are timestamps. After some enumeration, you'll find an empty folder at **/opt/race/**

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/1a80a178-3771-4a27-8323-9bec6caa8d95)

There's another race going on here. Deleting vs Executing. Let's start by building an exploit, throw it inside **/opt/race** and run **sudo /root/leclerc**. Hopefully the exploit (setting SUID bit for /bin/bash) wins the race and is executed.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/dcab8cda-7907-42c0-b875-e6cb6e75eb70)


Thanks for reading

Bruno Caseiro
