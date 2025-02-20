---
layout: single
title:  "Girls don't pay and neither do guys"
header:
  overlay_image: /assets/images/banner.jpg
  show_overlay_excerpt: false
---

## Introduction

I'm not a party guy, but I think we've all wondered what's up with girls paying less than guys... but that's a conversation for another day, because that won't be a problem anymore.

Now, everyone pays the same: nothing!

This will be a short post about a pretty cool vulnerability I found on a web application that sells tickets for college parties in Aveiro. Before you ask, there is no vulnerability disclosure program but I'm a professor there, so technically I already work for the University. 

It's not an extremely technical vulnerability, but it was one of those simple but exciting "a-ha!" moments. It's perfect to show to students and newbies in cybersecurity since I don't think it's very hard to understand, you just have to be extra creative. That comes with practice and experience.


## Exploring the application

This is a pretty straightforward app. You have a tab for your profile and settings, which we don't really care about right now. Here's the wallet tab, notice my balance is at 0€:

<p align="center"> <img src="https://github.com/user-attachments/assets/001ea170-f381-480e-af61-c36484d23bfb"> </p>

The tickets tab, showing that I have not purchased any tickets:

<p align="center"> <img src="https://github.com/user-attachments/assets/64a44efe-0451-4f08-beb2-ba57de33b264"> </p>

And the active events list. At the time I took the screenshots, tickets were being sold for the valentine's day party on the 13th:

<p align="center"> <img src="https://github.com/user-attachments/assets/a64c1228-2c62-4bb1-9f5c-3c2201f617dc"> </p>

One more introduction before we just into the fun stuff. If we try to buy a ticket, the "Wallet" option is unavailable since we have 0€. The complete flow isn't shown but there's nothing interesting, just a standard add-to-cart flow.

<p align="center"> <img src="https://github.com/user-attachments/assets/688ce004-ff75-4ded-b33c-e41e7aade4eb"> </p>


## The fun stuff

I started messing around with the buying flow and looking at the requests. Initially I just tried to set the "qtd" parameter to 0 on this request, hoping that it would only affect the price - same number of tickets for the price of 0 tickets.

<p align="center"> <img src="https://github.com/user-attachments/assets/ac16e4a6-b1ed-4bb6-945f-ae9196b78eef"> </p>

It turns out the price did change, I'm not sure if the number of tickets did too, but either way this is a dead end. I cannot use my Wallet. And I won't touch the other two methods since they're external components, developed by third parties.

<p align="center"> <img src="https://github.com/user-attachments/assets/687484dd-5a00-4f3c-a313-0af8928f08c0"> </p>

Okay, let's go for a second attempt. What if I change the "qtd" parameter to a negative number? Surely the application will be ready for this...

<p align="center"> <img src="https://github.com/user-attachments/assets/0de9efa7-e3c8-49b0-b20d-a55b607192eb"> </p>

This is where everything gets weird. I can now use my wallet. The total price is -4.00€ (one ticket was priced at 4€). It actually accepted the negative value. And notice how I can pay with the wallet again.

<p align="center"> <img src="https://github.com/user-attachments/assets/bd06af9f-32be-47d0-bcda-9f13eec69d99"> </p>

What I think is happening here is that the application allows paying with the wallet since WalletBalance > CartPrice. This is probably one of the first few checks because earlier we tried to buy 0 tickets, and the wallet didn't show. This is likely a combination of weak and unordered security checks.

For example, let's say you check the following things: 1) Is the wallet's balance > 0€? If not, you can't buy; 2) Is the wallet's balance greater than the cart's balance? If not, you can't buy. If yes, proceed.

This would work. Now let's say the checks are performed in the inverse order. It first checks if my wallet's balance (0€) is greater than the cart's balance (-4€). It is, so I proceed without ever checking if my wallet has more than 0€. I can't say for sure this is the logic behind the application, but it must be something like this.

I got over the wallet not displaying problem, and I noticed another request being sent... Again, the "qtd" parameter is being set with the value -1.

<p align="center"> <img src="https://github.com/user-attachments/assets/154702aa-7190-4e59-ad85-ab80918c0a81"> </p>


Let's leave it like that for now... The application displays a message - "Ticket bought successfully". But... I still have no tickets. My guess is that the application is trying to give me -1 tickets.

<p align="center"> <img src="https://github.com/user-attachments/assets/fb0b85db-f043-45af-b41e-2a6c07121080"> </p>

Okay, cool. We haven't really exploited anything, but we're figuring how things work. Just because we don't have a finding yet, doesn't mean we're not making progress.


## Okay sorry, time for the actual _fun stuff_

Let's do the exact same flow - try to buy a ticket, change the "qtd" parameter on that first request so the wallet is available to us (0€ > -4€ so it let's us use the wallet) and intercept that second request.

It turns out this request control the number of tickets we will be getting. So we can just change it to 1.

<p align="center"> <img src="https://github.com/user-attachments/assets/8f7d86ab-7ce1-4ee2-bad0-46aa5c29ed40"> </p>

In sum, we paid the price of -1 tickets for 1 ticket. Here it is, in our tickets tab!

<p align="center"> <img src="https://github.com/user-attachments/assets/badd63ae-39c9-4a9b-91aa-044ffe1aede0"> </p>

If I press the ticket, it shows me the QR code scanned at the entrance.

<p align="center"> <img src="https://github.com/user-attachments/assets/2d36d4ff-76e9-4f56-bd3a-34e938e2f58f"> </p>


## The end

As I mentioned, I'm not a party guy, but I did want to make sure this worked. Let's say someone might or might not have gotten into a party for free once or twice. Perks of being a hacker's friend.

Jokes aside, this was reported and fixed pretty quickly. The quantity parameters are now validated on server side and will throw an error if you try to get cute with them, so don't bother trying to smuggle yourself into college parties.

Here's me on 1.25x speed trying to explain this in video format. I did mix things up a bit with the logic of the payments so there's a little note halfway through the video. In my defense, that was in one take!

<p align="center"> <video width="50%" controls> <source src="https://github.com/BrunoCaseiro/brunocaseiro.github.io/raw/refs/heads/master/_posts/poc.mp4" type="video/mp4"></video></p>

