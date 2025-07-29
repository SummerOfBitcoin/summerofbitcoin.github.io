---
layout: post
title: "My Summer of Bitcoin Journey: An Introduction to Open Source and LDK"
date: 2024-08-09
author: "Ian Slane"
categories: [Stories, LDK]
---

It’s been awhile! After not posting during the proposal round, I’m excited to say that I’ve accepted an internship with the Lightning Development Kit (LDK)! After dedicating a considerable amount of time and effort to my project proposal and the corresponding code challenge, I’m now part of an amazing team, and I feel ready to share more about the project I’m working on this summer!

### I GOT THE INTERNSHIP

Securing this internship was no small feat. The selection process required us to complete a code challenge with a minimum passing score of 60% and the submission of up to three project proposals. The challenge involved writing a script that processes a series of transactions, validates them, and then mines them into a block. Each component, from transaction validation to block mining, had to be meticulously developed. The code challenge turned out to be one of the most complex pieces of code I’ve ever written, and I’m incredibly proud of it (with a score of 88!). I had to manage this all while being a full-time student as well, so it ended up taking me 24 whole days to achieve a passing grade, leaving me with just 6 days to finish a proposal. Given the time constraints, I just focused my efforts on a single proposal: adding BIP21 Unified QR Code Support to LDK-Node!

### WHY LDK?

I chose this project because LDK is a software development kit, and to me, there’s no better way to understand the underlying infrastructure than by contributing directly to it. The project aims to integrate Unified QR code support into LDK-Node, using LDK’s capabilities to streamline both on-chain and Lightning payments. This upgrade simplifies payment options into a single QR code, improving user experience and promoting broader adoption. My interest in Lightning and its potential was a huge motivator in choosing this project, as was the opportunity to work on something that bridges my understanding of transactions with real-world applications through payment URIs and QR codes!

### PROJECT OVERVIEW

The project involves utilizing existing LDK-Node payment handlers and their APIs to retrieve on-chain addresses, BOLT11 invoices, BOLT12 offers, and keysend payments. These elements will be passed into helper methods I make to construct a URI using the bip21 Rust crate. Then vis vera, I’ll have a method to parse a URI string and pay the invoice, and if that fails pay the on-chain transaction.

### IMPORTANCE OF THE PROJECT

This project addresses a critical need for simplicity in bitcoin payments. By integrating both on-chain and Lightning payment methods into a single QR code, we eliminate the confusion users face when dealing with multiple payment types. Currently, if a user scans a Lightning invoice with an on-chain wallet, it results in some annoying error. However, Unified QR codes resolve this issue by ensuring that wallets can automatically choose the appropriate payment method. This seamless experience is ideal in promoting wider adoption and user satisfaction.

### PROJECT GOALS

My primary goals for the project are:

1.  API Development: Finish and test the API for paying with BIP21 Unified QR codes.
2.  URI Generation: Finish and test the API for generating Unified QR code URIs.
3.  Integration: Integrate these APIs with LDK-Node bindings for Kotlin and Swift.

In addition to these project-specific goals, I have personal development objectives. I aim to improve my proficiency in Rust and Git. Rust’s emphasis on speed, safety, and reliability aligns with my interests and style of coding, and mastering Git commands is essential for effective contributions in any workplace!

### UP NEXT

I plan to document the rest of the summer through regular blog posts, covering different aspects of the project, including BIP21, BOLT11, design and architecture, implementation, and the creation of Swift and Kotlin bindings. This is only the first post, so stay tuned for the next one, where I will discuss the initial setup challenges and go into more detail about BIP21 and BOLT11.

Thanks for joining me! If you found this article insightful or have any questions or critiques, I’d love to hear from you. Feel free to reach out to me on Twitter or LinkedIn.
