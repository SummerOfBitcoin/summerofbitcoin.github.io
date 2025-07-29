---
layout: post
title: "My Summer of Bitcoin Journey: Building a Desktop App for Robosats"
date: 2024-07-25
author: "Amit Panwar"
categories: [Robosats, Stories]
image: "assets/images/avatar.png"
---

## A Passion for Bitcoin: From Curiosity to Commitment

I’ve always had a soft spot for Bitcoin — not just as a cryptocurrency but as
a revolutionary force in the financial world. My fascination with its
underlying technology and implications for privacy and decentralization led me
down a rabbit hole last year, where I invested time to deeply understand how
Bitcoin operates. Armed with newfound knowledge and a burning passion, I
decided to apply to the Summer of Bitcoin program. My proposal? A well-
thought-out plan that, despite my enthusiasm, didn’t make the cut. But hey,
setbacks are just setups for comebacks, right? The experience was a goldmine
of learning, further stoking my enthusiasm for Bitcoin.

## The Second Time’s the Charm: Project Selection and Proposal Writing

This year, when the Summer of Bitcoin opportunity knocked again, I was ready
to answer with a vengeance — or, at least, a well-prepared proposal. After
scouring through the available projects, I found one that felt like it was
calling my name: the Robosats project. With a strong background in JavaScript
and Node.js, this project was right in my wheelhouse.

Robosats offers a simple and private way to exchange Bitcoin for national
currencies. It’s like the James Bond of Bitcoin exchanges, using lightning
hold invoices to minimize custody and trust requirements. The platform even
gives users deterministically generated avatars to keep things private and
anonymous — no more awkward exchanges of selfies for ID verification! My
proposal aimed to elevate this platform by developing a desktop application
using ElectronJS, making Robosats more accessible and user-friendly.

## Making Waves: Project Progress and Achievements

Once the project kicked off, I dove in headfirst. Here’s a quick rundown of
the milestones I’ve hit so far:

1\. ElectronJS Compatibility:  
I got the Robosats web application cozying up with ElectronJS, transforming it
into a full-fledged desktop application. This involved reworking the web app’s
structure and functionalities to make it play nice in an ElectronJS
environment.

2\. Tor Proxy Integration:  
Privacy is the name of the game in the Bitcoin world, and I wasn’t about to
let Robosats fall short. I integrated Tor proxy support into the application,
ensuring all user interactions are cloaked in anonymity. Think of it as
putting the whole app in a stealth mode — no one’s tracking your moves here!

3\. Storage Security Enhancement:  
When it comes to user data, security isn’t just a feature; it’s a must-have. I
upgraded the client-side storage from `localStorage` to `sessionStorage`. Why?
Because sensitive data shouldn’t be lying around longer than necessary. This
tweak means enhanced security, reducing the risk of data leaks or unauthorized
access.

4\. Automated Builds with GitHub Workflow:  
Who doesn’t love automation? I implemented a GitHub workflow to automatically
build the desktop application for all major platforms — Windows, Linux, and
Mac. Now, the latest versions are built and ready to deploy with minimal fuss,
making the development and release process smoother than ever.

These enhancements are more than just code updates — they’re steps towards
making Robosats a stronger, more secure, and user-friendly platform. I’m
thrilled with the progress so far and can’t wait to see where this journey
takes me next. After all, when it comes to Bitcoin, there’s always more to
learn and build!
