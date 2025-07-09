---
layout: post
title: My Summer of Bitcoin experience with BDK
author: Pedro Felix
date: "2022-08-25 11:06:05 +0000"
tags:
  - "Stories"
---

## What is Summer Of Bitcoin?

SOB is an online internship for university students focused on bitcoin open-source projects. Students who get selected get paired with a mentor who provides quality resources, helps them navigate the world of open source bitcoin programming, and has the crucial role of guiding the mentee towards completing their first contribution. SOB provides seminars, free shipped goodies and it's a paid internship with the option of getting the stipend in Bitcoin.

## What is Bitcoin Dev Kit?

Bitcoin Dev Kit aka BDK is an open-source library used to create desktop, mobile and web assembly (WASM) Bitcoin descriptor based wallets. BDKs library is written in Rust and enables anyone to build custom cross-platform Bitcoin wallets. BDK has language bindings through which the BDK library is accessible via API to develop with Kotlin/Java(Android), Python, and Swift(iOS). With BDK you don't have to reinvent the wheel or start from scratch, you can use BDK as a standard library to build your own wallet. If you want customization BDK is modular which means you can modify wallet features, have a custom blockchain client, database, coin-selection and more. BDK uses descriptors which allow managing complex descriptor-output wallets with ease.

# My experience

I was privileged enough to work with the Bitcoin Dev Kit in the sub-project named `bdk-ffi` alongside mentorship from Steve Myers @notmandatory who is a full-time bitcoin contributor. `bdk-ffi` is a language bindings generator for the Bitcoin Dev Kit Library. In other words this FFI (foreign function interface) allows access to the BDK Library as an API to be used by native languages such as java, swift, and python. The bdk-ffi project enables users to program in their native language without having to use Rust directly.

### How I started

First I started by installing all the required development tools like git, Rust, an IDE (Visual Code Studio), and git cloned the `bdk` and `bdk-ffi` project repositories to my local system. I skimmed through the bdk documentation and started getting familiar with the section of the code in which I was going to work on. I read and skimmed through most of the documentation from [Rust](https://doc.rust-lang.org/beta/?ref=blog.summerofbitcoin.org), [UNIFFI-RS](https://mozilla.github.io/uniffi-rs/?ref=blog.summerofbitcoin.org) and the [BDK](https://docs.rs/bdk/latest/bdk/?ref=blog.summerofbitcoin.org) docs. I went over Rust standard library documentation, read the part of the Rust book and looked up video tutorials online to learn Rust as quickly as possible. [Rustlings](https://github.com/rust-lang/rustlings?ref=blog.summerofbitcoin.org) is another good program that teaches how to use Rust interactively. When I needed to review concepts I used the "[Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook?ref=blog.summerofbitcoin.org)" book as reference by Andreas M. Antonopoulos. BDK has a [Discord](https://discord.gg/dstn4dQ?ref=blog.summerofbitcoin.org) server which is open for discussion and questions related to BDK development. Many of the conversations that don't take place on github take place on discord. The BDK team has regular team calls open to everyone which are perfect to learn about the latest project updates and discuss work in progress.  
I started working with the example code and the process of just running the code and try to see what was happening.  
I also read documentation, looked up tutorials and did online research before much of this made sense. There were many days I felt like zero progress was made, Bitcoin is challenging intellectually. Since Bitcoin has a lot of surface area to cover it quickly became addicting trying to figure out what was going under the hood, more and more question to be answered.

### What I did

Working on my project required that I copy the repository of my project locally. I worked on the project locally in Visual Code Studio an IDE and started to get a sense of what is happening in the code and how it works. It was a bit daunting at first to go into the code and not understand much. But as I put in the effort and time, this started to become more clear and get easier with time. By studying closed PR's and using them as example code I was getting an idea of how the code worked. I wrote notes and kept a journal of the things I was working on something I think is really important.

### First PR merged

Before I was able to even start coding, I spent a lot of my time reading documentation, running example code, and setting up before I was successful in merging my first code change. Before the code change I researched BIP-74 which is a Bitcoin OP\_RETURN which enables you to write arbitrary data on the blockchain. My job was to enable this feature for the bdk-ffi library which handles the language bindings in Rust. This feature was already in the main bdk library but what I had to do was enable and implement it into the language bindings FFI aka foreign function interface. The bdk-ffi works as an API wrapper for the bdk library.

### My first code test review

I manually tested some code and made comments and code change suggestions on an open PR that had no reviews yet. I found it challenging to review someone elses code because If you are not familiar with the problem, it might take a while to first understand what are the changes being made. I later manually tested the code changes by running the program locally and running all the commands possible that I could access to test functionality. By testing the open PR I assisted in helping find a small bug in the program. Although testing could be seen as something small, it is something much needed in the bitcoin ecosystem right now. There is a lot of code pull requests (PRs) that need to be reviewed and not many reviewers.

#### What is still to do

After Summer of Bitcoin I still have two PR's waiting to be merged. One of the PR's is ready to be rebased and then it will be ready to merge. The second PR was ready, but since some major code changes were made to the library I will need to wait into that PR is merged, so I can rebase on top of that and finish the PR. Summer of Bitcoin has been an awesome experience, I've learned more than I thought I was able to handle. My plans are to continue monitoring the bdk project closely and continue to work with them. I've gotten a glimpse of what is possible with Bitcoin and open-source software, I couldn't be more optimistic about the future.

#### Suggestions for Future Contributors

Summer of Bitcoin is an awesome summer experience because the tools, community, mentors you are provided with make the learning curve of tackling bitcoin development easier to climb versus going it alone. They’ve laid down the foundation and tools you need to succeed in learning about Bitcoin. I’m going to outline some of the skills I learned in one summer through this amazing intellectual program.

* I've learned a new programming language and various libraries that are great to learn for any software developer.
* I learned how to use an IDE for better work flow and faster learning.
* I learned how to use the basics of Git, Rust, Cargo and refined my note taking skills.
* I learned not to waste too much time on a problem, it’s okay to ask , sometimes its not worth being stuck on a problem for hours.
* Programming language Rust which is super cool! If you know some C++ I think you will catch on pretty quick depending on your programming experience.

## Reasons why I enjoyed the internship.

* I contributed on my first PR on an OSS project by creating and reviewing PRs that were eventually merged.
* I was exposed to many of the technical details about Bitcoin.
* I was able to develop new skills, learn a language named Rust.
* I heard about many new ideas emerging in the Bitcoin space.
* I worked with people who are passionate about their work and learned from them.
* I got to wake up every day excited about going to work on Bitcoin.
* I was connected to other developers currently working on Bitcoin projects who were more than happy on guiding me when I asked questions.
* I was able to increase my Bitcoin knowledge from seminars, peers, and the work you put into the program.
* I was able to stack some sats while learning about Bitcoin.
* I now have experience and can keep contributing and further develop my skills.

## Notes for First Time Contributors

Although sometimes some code changes may be relatively easy depending on the type of implementation needed, Bitcoin isn't easy don't be discouraged, keep practicing, and work hard. An option is to look at closed PR's to see how previous code merges were done. Study the code file changes and see how it was implemented. This way you learn from others quickly rather than figure everything on your own. Review peoples code before you create your own PR or look for what they call "low hanging fruit" which can be first-time-contribution PR's or easy ones. Before asking for help make sure you have used your online resources and thought out the question well.

## Conclusion

Summer of Bitcoin was the best investment of time I could have made this summer without a doubt. I'm ready to keep learning and contributing. This is just the beginning of what's to come in my Bitcoin career, but for now I'll be working on BDK on the side while I go to school and keep learning about Bitcoin. I have my eyes on [LDK](https://lightningdevkit.org/?ref=blog.summerofbitcoin.org) (Lightning Dev Kit) :eyes: another awesome Bitcoin open source project with a Lightning node implementation.

## Thank Yous and Credits

* Thank you, Adi for this amazing summer program where I was able to develop some skills, knowledge, and valuable experience. I look forward to learning more from you and seeing you around the Bitcoin space.
* I want to thank my amazing mentor Steve @notmandatory for being a patient, hard working, helpful and amazing mentor. If it wasn't for Steve's guidance I don't think I would of learned as much as I did in this internship. If anybody is lucky enough to be mentored by Steve you know he really has a way of breaking things down easily and explain things easily with skillful articulation.
* I also want to thank Summer of Bitcoin for hosting this amazing program and everyone else involved.
* Thank you, Bitcoin Dev Kit community for this amazing project.
* Thank you Spiral.
