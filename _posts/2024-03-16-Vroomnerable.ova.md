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
- <https://drive.google.com/file/d/12diIlN_pxzt94d1ysCSYpj7AxJKGR8xf/view?usp=sharing>




It's the first time I'm building a vulnerable VM, so I'm not expecting it to be perfect. But come on, give it a go! Write a walkthrough or give me your feedback. Don't forget to ping me!






# SPOILERS AHEAD

## Walkthrough

### Setting up
If you're using VirtualBox, importing the machine should be pretty straight forward after downloading the .ova file. Simple press **File --> Import appliance** and select the downloaded file. It should come pre-configured (I recommend setting your machine and the vulnerable machine to "NAT network", that never results in any network problems for me).

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/3d4bacea-3c6a-4c1b-8b6a-0bedd0582ae1)


You boot both machines (your Kali and vroomnerable) and if everything goes according to plan, they should be on the same network. Time to find out the victim's IP address.
The attacker's IP is **10.0.2.4/24**, so **vroomnerable** should be somewhere in the **10.0.2.0/24** range.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/74f0e7e5-61c3-4b54-9ad3-b2868b1632f7)

To be sure which of the results is the target machine, you can compare the MAC addresses found to the VM's MAC address in the network settings.

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

The source code is in the home folder. This is artificially vulnerable due to the **log_operation()** function. The larger the value in the for loop, the longer the thread will wait before setting a value. **wait_set()** sets a PENDING value, while **set()** sets an ACTIVE value.
When sending a lower value, the **else** branch is taken and the artificial delay starts. That's the race condition window and when the second value (150) should change the y variable before the first thread completes.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/49071ab0-bb56-4aa0-ad09-8646af1333d3)

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/de14ede0-5f9c-48e5-8d99-3b4ca2102bf8)


### Root flag
This is where it can get tricky. At first, this is the only thing we have to escalate our privileges.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/6ab8d385-be0a-41bf-8447-01854371cb89)

Some files are being deleted and some files are being executed. The numbers are timestamps. After some enumeration, you'll find an empty folder at **/opt/race/** - this is where the deleting and executing is happening.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/1a80a178-3771-4a27-8323-9bec6caa8d95)

There's another race going on here. Deleting vs Executing. Let's start by building an exploit, throw it inside **/opt/race** and run **sudo /root/leclerc**. Hopefully the exploit (setting the SUID bit for /bin/bash) wins the race and is executed.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/dcab8cda-7907-42c0-b875-e6cb6e75eb70)

I guess we came in second. The goal here is to extend the deleting window enough so that all the executions can happen inside it. Please allow my artistic (aka paint) skills to demonstrate that.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/c0125e6e-1e4c-4174-bfc4-3e0608978cce)

However, this can only happen if deleting files takes longer than executing files. Let's put on our scientific goggles and do an experiment. Here's what we're going to do:

1) Copy the exploit 10 times to **/opt/race/**
 
2) Run **sudo /root/leclerc**
 
3) Collect the length of the executing and deleting windows
 
4) Repeat steps 1-3 with more files each time

In practice...

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/8c8cfc0f-6b00-4989-8a0b-7bc7b5a71f3d)

Here are the experiment results:

| **Files**  | **Started deleting files** | **Finished deleting files** | **Deletion Δ** | **Started Executing files** | **Finished executing files** | **Execution Δ** |
| --- | --- | --- | --- | --- | --- | --- |
| 1  | 1710590071065 | 1710590071066 | 1 | 1710590071065 | 1710590071067 | 2 |
| 10  | 1710591111140 | 1710591111142 | 2 | 1710591111140 | 1710591111148 | 8 |
| 100  | 1710591215249 | 1710591215266 | 17 | 1710591215249 | 1710591215335 | 86 |
| 1'000  | 1710591235234 | 1710591235399 | 165 | 1710591235234 | 1710591235851 | 617 |
| 10'000  | 1710591263709 | 1710591265325 | 1'616 | 1710591263709 | 1710591265644 | 1'935 |
| 30'000  | 1710592360610 | 1710592365380 | 4'770 | 1710592360610 | 1710592365814 | 5'204 |
| 60'000  | 1710592621064 | 1710592631090 | 10'025 | 1710592621065 | 1710592631257 | 10'192 |
| 90'000  | 1710592938159 | 1710592953692 | 15'533 | 1710592938159 | 1710592953983 | 15'824 |
| 200'000  | 1710594998488 | 1710595035416 | 36'928 | 1710594998488 | 1710595035422 | 36'934 |
| 300'000  | 1710595954811 | 1710596010562 | 55'751 | 1710595954811 | 1710596010571 | 55'760 |

For a very large number of files, the execution delta will eventually be closer and closer to the deleting delta. I didn't try with more than 300k files as the copy takes a long time (I'm sure there are faster ways to do it) and also because at this point, my exploit was actually executed.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/ec533daa-d23a-48fe-a51e-89b4ab38580e)

I left the source code in the **/root/** folder, inside the file called **lecler.c**

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/11b7655f-6553-48e2-96d9-cff075fba1c9)

The main function simply starts two threads almost simultaneously - one deleting files and another one executing files, both in the **/opt/race/** directory.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/6c0b1cb2-f8ad-4865-827b-6d20a6555435)


The vulnerability is in the **delete_files()** function. I addedd a very short sleep, which after a bunch of loops will accumulate and create a considerable delay. 9 microseconds was the value that would delay the program just enough to be able to exploit it with at least 100k files (might take some tries).

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/2fe80f35-4ec0-4dcf-a79c-9026d5275ce6)

Still, my paint masterpiece above isn't 100% in line with the experiment results. My theory is that **execute_files()** finishes faster than **delete_files()**. What I mean is that all files are executed and immediately after, the timestamp is printed.

This does not happen with **delete_files()**, which is a bit more robust - there are error checks and the call to **closedir(dir);** before printing the timestamp. Maybe that takes at least 9 microseconds, which would explain how the exploit worked for 300k files even though the deleting delta is shorter than the execution delta. All this math only works if the starting timestamps for both operations are the same, which is in fact the case.

![image](https://github.com/BrunoCaseiro/brunocaseiro.github.io/assets/38294180/bdff30d8-a919-4e4e-9cd2-1c558f29c3fd)
