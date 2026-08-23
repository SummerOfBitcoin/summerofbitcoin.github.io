---
layout: post
title: "One Relay for Every OS: Breaking strfry Out of Linux"
date: 2026-08-23
author: Shivansh Jain
categories: [strfry, Nostr, Stories]
image: ../assets/images/8.jpg
---

# One Relay for Every OS: Breaking strfry Out of Linux

This wasn't my first time trying for Summer of Bitcoin. I participated last year, did all the assignments, but academic exams blocked me from submitting a proposal on time. But this year? This year was different. I found the right organization, a project that clicked with me, and a highly active maintainer in Doug Hoyte.

I am Shivansh Jain, a 3rd-year B.Tech CS student at IIIT Lucknow, and this is the story of my Summer of Bitcoin journey contributing to `strfry`.

## The Warm-up: Force Pushes and Proposals

Before the official period began, I was actively contributing. Some PRs got merged, others stalled out. I will be honest - I pushed a lot of wrong code. I would test it, realize the fix did not actually fix the issue, write another fix, and shamelessly force push over the wrong code. We have all been there.

The proposal phase was intense. I wrote mine alongside a friend who also got selected. We spent days tearing apart each other's drafts to make them better right up to the deadline. I was actually too late to get my proposal reviewed by the mentor before submission. But it worked out, largely thanks to teaming up with Anurag. Since I was new to low-level C++ and production-grade applications, I asked if he wanted to collaborate on the Performance Optimization project. Surprisingly, both the maintainer and Anurag agreed to the co-contribution model.

## Mac Compatibility: Remote Shells and Refactoring

Initially, I tried tackling random `FIXME` and `todo` tags scattered around the codebase. But as the SoB period kicked off, I gravitated toward a bigger problem: cross-platform compatibility.

At the time, `strfry` only supported Linux and FreeBSD. I wanted to bring it to Mac and Windows, knowing that Mac support is critical since a huge chunk of developers use it daily.

There was just one problem - I did not own a Mac.

I ended up borrowing my friend's machine. I discovered a tool called `mutagen` that let me sync the codebase seamlessly. He would keep his Mac open, going about his day, while I accessed his shell via SSH. I could write code on my PC, sync it, and compile it on his Mac with practically zero lag.

The actual code changes were a learning curve. The file watch monitor required PRs across multiple repositories since `strfry` relies heavily on submodules. My initial approach worked, but it relied heavily on `#ifdef` preprocessor directives, which made the code incredibly hard to read. Doug suggested I use proper abstraction and divide the platform-specific logic into separate files. The refactored result was significantly cleaner. It was one of those "aha" moments that made the whole experience incredible.

## Benchmarking the Beast

When I wasn't wrestling with OS support, I was trying to build a benchmarking system. The architecture decisions were tough: do I write a script, or create a custom binary for stress testing?

I decided to do both: scripts for the logic and a custom binary to apply real pressure. I scoured Google, ChatGPT, and Gemini for metrics to track. The biggest hurdle was hardware dependency. A relay performs better on better hardware, making absolute numbers meaningless across different machines.

We decided the best approach was a relative benchmark: run it once on `master`, and once on the PR branch. I considered automating this via CI, but CI runners scale dynamically based on load, meaning you never get the same base specs twice. The downside? Testing became painfully slow. A single run could easily take 30 to 60 minutes on a good PC. I ended up keeping a lot of these changes local while I figured out the edge cases.

## Submodule Hell: Native Windows Support

Next up was native Windows support. The goal was to compile `strfry` using the MSYS2 environment so it could run natively globally on Windows.

The code changes were not massive, but the repository structure was. I had to touch five different submodules: `golpe`, and inside that, `hoytech-cpp`, `loguru`, `rasgueadb`, and `config`.

Here is the catch: `config` and `loguru` were upstream dependencies maintained by completely different people. We had to submit PRs, wait for them to get merged, and then update the submodule pointers one by one. It was a massive headache, but honestly, I came to appreciate the intricate submodule structure. I had seen submodules before, but never utilized this deeply in production.

## The College Balancing Act

I wanted to do so much more, but reality hit hard. College duties left almost no breathing room.

Right after exams, our initial internship placement season began. My days devolved into revising academics, grinding problem-solving, and studying system design. Just as that cooled down, Technical Club selections started. Being in my 3rd year, I was interviewed and selected as the FOSS Wing Coordinator and a senior member of the Web Wing. That meant I was now the one conducting interviews for the junior batch.

If my time management had been better, I could have balanced SoB and college more gracefully. It is definitely something I need to work on, along with my communication. I somewhat regret not communicating as consistently with Doug as my peer Anurag did. That level of professional sync is something I am aiming for.

## Signing Off

Looking ahead, I plan to stick with `strfry` whenever I find the time - whether that means continuing my Windows compatibility support or helping onboard new contributors.

I am also eyeing more open-source programs like GSoC. I tried it this year but could not maintain the consistency needed. But this summer taught me the real formula for open source: communication + proof of work + a bit of luck. That is all you need.

Good luck to future Summer of Bitcoin applicants, or anyone just reading this out of curiosity.

ThunderBlaze Signing off
