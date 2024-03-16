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


Thanks for reading

Bruno Caseiro
