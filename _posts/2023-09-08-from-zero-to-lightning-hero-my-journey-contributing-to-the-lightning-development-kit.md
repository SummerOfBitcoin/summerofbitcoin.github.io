---
layout: post
title: "Demystifying the Lightning Development Kit: A Beginner's Guide to Contributing"
date: 2023-09-08
author: Prakhar Saxena
categories: ['Stories', 'Lightning']
---

In this blog, I will talk about my experience in contributing to **LDK** and
how a beginner contributer can get started.

# LightningDevKit

Lightning Development Kit is a library that allows you to build a lightning
node without worrying about implementing low-level lightning logic correctly.  
  

![LDK Logo](/assets/img/bitcoin/ldklogo.png)

  
  
**LDK** is based on Rust-Lightning, a full-featured but also incredibly
flexible lightning implementation designed to be modular and customizable. It
allows you to build a lightning node without having to worry about all of the
low-level lightning logic like the state machine or routing.

There are a lot of repositories available on the LDK’s Github. These include:

  * Rust-Lightning \- It is a generic library which allows you to build a Lightning node without needing to worry about getting all of the Lightning state machine, routing, and on-chain punishment code (and other chain interactions) exactly correct. This is also the core repo of LDK.
  * LDK C Bindings \- Main LDK C bindings on which other bindings are built
  * LDK Garbage Collected \- LDK bindings for garbage-collected languages e.g Java
  * LDK Swift \- LDK Swift bindings for iOS
  * LDK Sample \- Sample node implementation using LDK
  * LDK Documentation \- LDK’s open source documentation   
  

# Pre-Requisites

These are some Pre-Requisites that might help a new contributer in
contributing to any lightning network/bitcoin project.

  * Basic knowledge of how bitcoin works(Mastering Bitcoin)
  * Basic knowledge of how lightning network works(Mastering the Lightning Network)
  * Basic Rust experience and knowledge(First 7-8 chapters of Rust Lang Book)   
  

# My Experience

Since the initial Summer of Bitcoin phase, I was interested in Lightning
Network and LDK seemed to be the perfect organisation to contribute to.
However, the core repo of LDK(Rust-Lightning) uses Rust which I didn’t know
back then. So I started by learning Rust. I went through the first 8 chapters
of the Rust Lang Book.  
  
The Summer of Bitcoin organisers also encouraged new developers to learn Rust
and created discord channels for doubt solving. After having some basic grasp
on Rust, I looked into `good first issue` in the Rust-Lightning issues list
and commented on the issues. The developers were really friendly and helped me
through the entire issue.  
  
The LDK Discord is a really nice place to ask questions. Even experienced
developers(Matt Corallo, Jeffrey Czyz, etc.) take out their time and encourage
questions. That being said, it is always good to ask intelligent questions.
This can be achieved by referring to the Lightning Network/Rust books and
doing good research on your part before asking the developers.  

# Contributing to LDK

The best place to start would be LDK’s Contributing guide. It has got detailed
instructions on how to start contributing to LDK along with workflow details
and guidelines.  
Going through LDK’s Documentation can also be a good starting point if you
want to learn about LDK in detail. There are many ways to contribute to LDK:

  * Documentation - If while going through the documentation you find a typo or have suggestions, the LDK discord can be used to discuss it and issues can be opened.
  * Good First Issues - These are relatively easy to solve issues and the LDK developers always encourage new contributors.
  * Peer Review - This involve reviewing the Pull Requests made by other developers for errors/suggestions. Pull Requests should be reviewed first on the conceptual level before focusing on code style or grammar fixes.
  * Testing - Writing new tests is heavily encouraged as it will help with improving security and may disclose some vulnerablities.
  * To Dos - There are some `To Dos` in the LDK codebase. These can be picked up by developers after discussion in the discord channel.   

Communication about the development of LDK and `Rust-Lightning` happens
primarily on the LDK Discord in the `#ldk-dev` channel. Additionally, live LDK
devevelopment meetings are held every other Monday 19:00 UTC in the LDK Dev
Jitsi Meeting Room. Upcoming events can be found in the LDK calendar.  
Contributors starting out with the Rust language are welcome to discuss and
ask for help in the `#rust-101` channel on LDK Discord.
