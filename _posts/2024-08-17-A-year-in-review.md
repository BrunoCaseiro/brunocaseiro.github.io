---
layout: single
title:  "A year in review - lessons learned"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

## Introduction

Nope, it's not the most wonderful time of the year yet. It's late August. I do love Christmas though, especially the weeks leading up to it. December is always so magical! But we're not there yet. It's actually almost my work anniversary - the day I went from Penetration Tester to Security Testing Engineer. Well, titles... I still do plenty of penetration tests, but I'd say my day to day revolves around Application Security Engineering now.

<p align="center"> <img src="https://www.explainxkcd.com/wiki/images/0/00/wrapping_paper.png"> </p>

I'm hoping I can keep a log of my adventures throughout my career. In a few years I'm sure it will be fun to laugh about whatever dumb things I write today. Additionally, someone else can take a peek and grab some ideas for personal projects, interesting ideas to implement in their organization, or anything in between. ~~<sup>jk, it's just a list of reasons i made up to ask for a raise</sup>~~

Anyway, here are some things I learned or that stood out over the last 12 months. I don't want to get too technical, so I'll keep this kinda vague 🙂


## Index
[1) Mentality Shift](#mentality-shift)

[2) Newcomer's guide](#newcomers-guide)

[3) Automating Jira-Slack alerts](#automating-jira-slack-alerts)

[4) Updating Active Directory documentation](#updating-active-directory-documentation)

[5) Race condition and the birth of this blog](#race-condition-and-the-birth-of-this-blog)

[6) Version control system "pentest"](#version-control-system-pentest)

[7) PCI Compliance](#pci-compliance)

[8) Certifications](#certifications)

[9) Starting an incident](#starting-an-incident)

[10) Going forward](#going-forward)

[11) The end](#the-end)


## Mentality Shift

One thing I'm particularly proud of during the last 12 months is how I've changed my mindset at work. I believe this is a useful tip that applies not just to professional organizations.

I used to be completely focused on my individual tasks, always trying to deliver the best work I can while learning as much as possible. Sure this is great, especially early in your career. But as you get more efficient, these daily tasks become second nature. When I felt comfortable enough at my role, I started looking for ways to improve other stuff slightly outside the scope of what's expected of me.

Well what exactly do I mean by this? Ask questions. I don't mean the usual "how do I do X" questions[^1], but question yourself about the processes, the methodology, the pain points. And don't restrict this to your own role and tasks (everyone loves a teammate/coworker that makes the others' jobs easier!). Ask yourself what could be improved or automated? What makes this task harder? Why does it take so long? Why is it done this way? Then try to find solutions. It's kind of like having an entrepreneurial mindset. I know, I know, it's a common buzzword, but seriously, it really makes sense in this context. 

<p align="center"> <img src="https://imgs.xkcd.com/comics/move_fast_and_break_things.png"> </p>

Aim to provide value to your team, to other teams and even to the organization as a whole. Experienced people might not necessarily do more work in terms of quantity, but they almost always carry bigger responsibilities and deliver work with greater impact.

Worried about looking stupid? Trust me, you're not alone. Be the one to ask the question. Even if it feels like a stupid question, that's okay. If you're honestly trying to learn or understand things, there will never be a stupid question. 

I felt that this attempt to be more proactive did not go unnoticed and eventually more opportunities and projects started coming my way. I do enjoy working on a wide variety of projects. I don't mean a high number of projects, but projects that are very different from one another. It creates plenty of learning opportunities and not just for purely technical knowledge. Sometimes you even get the chance to meet new people and teams inside the organization. Some of these were my own suggestions, but many came from others, especially my manager. If you're curious and eager to learn, things should start to come your way 🙂 

[^1]:Always try to solve the problem on your own! If you can't and really have to ask: be specific, provide context, explain what you've tried and the respective outcomes. Seniors have bigger responsibilities, so appreciate the time they're taking to help you! Also, read [no hello](https://nohello.net/). This is NOT an excuse to be rude. You can be polite and direct at the same time.


## Newcomers' guide

One of the very first things I did when I joined the company was to start documenting my days from a high-level perspective, eventually this ended up as a full documentation page for new Security Testing Engineers. And it's funny how this works, many questions I had are now super obvious to me and it only makes me think - "why did I even care about that?". This is similar to what happens in school. Sometimes teachers explain things in a super confusing way because they (unconsciously) assume you already know some stuff. Sometimes asking a colleague or a friend for a second explanation makes things so much clearer since you're both in the same "stage of learning".

Needless to say, this was reviewed by more experienced people to make sure I did not write anything stupid (ha! I now have a stupid blog and you can't stop me!). It's a good idea to share this document with newcomers, get their feedback and let them tweak it after a few weeks.


## Automating Jira-Slack alerts

One member of our team is always ready to provide support to other teams or to put out any fires that might arise. This role rotates every sprint, which is brilliant - it means not everyone has to be paying immediate attention to emails or Slack messages. Some of our tasks are extremely technical and require bursts of deep concentration, so these types of interruptions can get pretty annoying.

Anyway, teams often reached out to our Slack channel or just pinged whoever they felt like in the security testing team asking who they should contact about X or Y. Or they'd contact the person providing support from the previous sprint, not realizing it had been rotated. There's room for improvement on our side here.

I automated a Slack message through the Jira bot to announce when the support person changes. Each sprint the channel gets an alert like "Bruno Caseiro is available to provide support". Pretty simple at first sight, but it cut out one extra step to any team trying to get in touch with us.

<p align="center"> <img src="https://imgs.xkcd.com/comics/the_general_problem.png"> </p>


## Updating Active Directory documentation

I've been trying to learn a bit more about Active Directory, even running some assessments on our domains. I knew some basic concepts from the OSCP and CTFs, but never had the chance to actually conduct a real pentest. I got some pointers and notes from our Red Team and other more senior people and eventually put together a simple and easy to follow guide on assessing our domains. It includes specific tips to our organization, useful links, common commands, tool installation tips and so on. I've still got plenty to learn about AD, but I'm getting there!


## Race condition and the birth of this blog

<sup>"oh no, he's' gonna brag about that again!"</sup> No I'm not 😄 Buuuuuuut 🙄 Feel free to read about it [here](https://brunocaseiro.github.io/My-first-race-condition/). 

<p align="center"> <img src="https://qph.cf2.quoracdn.net/main-qimg-7d77f6f06368ff0d2ab29cf598ba5843"> </p>


## Version control system "pentest"

GitHub has a bug bounty program, that's not what I'm talking about here. The same way you don't find vulnerabilities in Active Directory itself, I was not supposed to find vulnerabilities in GitHub's products - what you commonly find during AD assessments is mostly poor configurations and other bad practices such as weak passwords.

Instead of domains, trees and forests, I had enterprises, organizations and repositories configured in many different ways. Just like during a red team assessment you can set some arbitrary goals, this pentest had some rough guidelines and objectives. Some were completed, others weren't. For a few of them, I was halfway there.<sup>jon bon jovi would be proud</sup>

I distinctly remember finding out about GitHub Enterprise's secrets. Organizations can set environment secrets available to "use" by everyone, like your OS's environmental variables. These might contain sensitive information so each accessible secret had to be analyzed on a case by case basis. You can read more about it on this [GitHub documentation page](https://docs.github.com/en/enterprise-cloud@latest/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions).


## PCI compliance

I did this for the first time this year and it was pretty interesting. PCI DSS stands for Payment Card Industry Data Security Standard. In short, if you handle clients' credit cards, there's a standard to ensure you're processing, storing and transmitting credit card info in a safe manner.

Part of this compliance test is in fact your typical web application pentest. Well, it would depend on your organization's use cases. But remember how I mentioned I liked doing a wide variety of things? A PCI compliance test covers that. Testing and reviewing network firewall rules, spotting credit card info flying around in unsafe ways and even checking if the SOC would respond in a timely manner to PCI-related alerts.

Not everyone can do these tests, so many times it has to be outsourced. If the company opts for an internal tester, as was the case, there must be proof that the tester is qualified enough to do it. Having the OSCP certification does the trick. I highly recommend volunteering for this if you have the chance. In fact, you should volunteer for anything that sounds weird or out of your usual scope of work. If it sounds foreign to you, there's absolutely no way you won't learn something new from it.


## Certifications

Speaking of the OSCP, one of the highlights of this year was the two certifications I obtained - the OSWP and the OSWE. Before diving into the specifics of each one, let me speak in a general sense first. The goal shouldn't be to obtain the certs themselves, but to master the contents taught in the courses. What's the point of having an alphabet soup on your CV if you don't use the skills? Sure, it might get you a few extra interviews because... well, ATS see alphabet soup ATS think good...[^2] but you're not getting past the initial rounds if you can't demonstrate that you're comfortable with the topics.

[^2]: Many companies use automation to filter candidates quickly in the early stages. Put yourself in the shoes of a hiring manager of a big company - sorting through thousands of applicants is tough, so don't take it personally.


### OSWP - OffSec Wireless Professional

I took this one first after studying for just a few weeks. There is a lot of theory in the materials, it's all pretty interesting but you can argue it's not very relevant in practical terms. It wasn't a hard certification, but it was definitely a lot of fun. One problem many people face is the lack of practice material. OffSec suggests you set up your own homelab which can get messy. No worries, I got you! [WiFi Challenge Lab](https://wifichallengelab.com/) - not every challenge is relevant for the OSWP, so look at the course materials and take an educated guess on what challenges from that website you should focus more 🙂

Lastly, and this goes for every certification or test you're going for, TAKE NOTES. Seriously, you see this piece of advice everywhere beCAUSE IT ACTUALLY WORKS. Take detailed notes on commands, payloads, and the workflow to exploit each type of WiFi security protocol. Trust me, you got this. It is not a hard certification.


### OSWE - OffSec Web Expert

The same cannot be said about the OSWE 🙃. It's an advanced, white-box pentesting certification, and you can clearly tell if you're going through the material. My approach was to read the PDF twice and also watch the videos twice. Take advantage of the practice machines provided by OffSec, they are there for a reason. AND TAKE LOTS OF NOTES.

The attacks taught are extremely complex and digging through the code to find the needle in the haystack can be a real pain. In the end, you'll need to automate the whole exploitation process - from the initial foothold in the web application to a fully interactive reverse shell. During the exam you might feel stuck for hours because you don't make evident progress. Unlike the OSCP, where you're constantly taking small steps towards the flag, in the OSWE you might take hours just familiarizing yourself with the code. This definitely will not feel like progress, but it is.

The exam was tough, but nothing beats the satisfaction of running a script and seeing that shell pop on its own a few seconds later. The best part is how useful the skills I've gained have been. My code reviewing and scripting skills have improved drastically. I used to feel extremely limited when trying to exploit vulnerabilities that required a bit more complex scripting or automation, but not anymore! 

<p align="center"> <img src="https://imgs.xkcd.com/comics/11th_grade.png"> </p>


## Starting an incident

Ever wondered what happens when poop hits the fan? (trying to keep it professional here, gents)

If you've never been there, there's usually several priority levels depending on how much poop there is. This level defines how urgently an issue needs to be addressed. Generally, higher severity = higher urgency = higher incident priority. During this past year, I've seen a few, luckily nothing major that caused a lot of disruption... Hmmm maybe due to the excellent job from all the security teams? 🙄

So, recently I found a vulnerability that required raising an incident. The concept of creating an incident isn't something very technical and I'm sure it varies from organization to organization, so there's no point in diving deep into the specifics. Anyway, it's great seeing different teams coming together and discussing different possible solutions. One thing I noticed from higher-ups was how their contributions aren't always extremely technical but nonetheless very, very valuable. Especially when it comes to risk analysis, for example - under what circumstances can this be exploited and how does it affect the company? A year ago I might have said something like "huh... no prepared statements = bad and passwords leaked!". That's not what I mean. Let's say an attacker would require a specific type of account which is only available to a small group of internal people. The risk immediately decreases. But not to zero! Internal threats are a thing too!

If you're in pentesting or AppSec, this is a lesson me and you can take advantage of. When writing reports always make sure to start a bit less technical and gradually go deeper into the details. Even the most experienced engineers can struggle understanding a vulnerability report if it jumps straight to the fun stuff without any context about the application or feature.

Bottom line: incidents aren't as stressful if you're breaking things rather than fixing them.

<p align="center"> <img src="https://languagelog.ldc.upenn.edu/myl/software_testing_day_2x.png"> </p>


## Going forward

This one is easy, I want continue in the Application Security field, broadly speaking. You probably noticed I enjoy doing a lot of different things, but there's one thing in common - the offensive security touch. I also want to improve my Active Directory skills and eventually some red team stuff, who knows. I might aim for the OSEP (OffSec Experienced Penetration tester) next. And then I'll be one step away from the OSCE<sup>3</sup>... If I manage to get the OSEP, there's no question I'll want to try and go for the OSED. It's definitely going to be one of the toughest challenges in my career, but I'm up for it 😄


## The end

"So it took you 12 months to do 9 things?" - Yes. Kidding, this isn't a diary, there's a ton of daily AppSec work in between. There's another thing that's kinda missing from this list. Another "unusual" project is cooking and I'm excited about it! 

And the blog lives on! This one's a bit less technical than the first two posts and feels more like burying a time capsule. Either way I'm hoping you can find a few knowledge nuggets in here. If you have any comments or disagree with anything I've said, feel free to let me know!
 
Time to ask ChatGPT to proofread this now...
