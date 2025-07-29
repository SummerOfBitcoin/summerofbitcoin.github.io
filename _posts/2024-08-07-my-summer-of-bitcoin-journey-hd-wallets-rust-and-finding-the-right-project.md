---
layout: post
title: "My Summer of Bitcoin Journey: HD Wallets, Rust, and Finding the Right Project"
date: 2024-08-07
author: "Ian Slane"
categories: [Stories, LDK]
---

Welcome back! As I continue with the third week of Summer of Bitcoin, I find myself reflecting on the whirlwind of experiences and insights gained so far. From the basics of bitcoin to unraveling the complexities of address generation and checksums , each step has been a extremely captivating. This week, I'm excited to share with you a recap of my experiences, talk about HD wallets as I understand them, and discuss the intriguing projects I've been researching. So, let's jump in and explore the latest chapter of my journey into bitcoin development! Let's get it!

### Summer of Bitcoin Week 3

Entering the third week, the depth of our breakout sessions seemed to deepen with each session. This time, our group talked about the complexities of bitcoin's HD wallets, sparking engaging discussions and exchanges of insights. Taking on a more active role within the group discussions, I found myself leading/co-leading without the formal designation, which only helped my confidence in navigating through the material. As time went on, I couldn't shake the anticipation building for the proposal round. It's exciting to see the potential collaborations and projects that come mid-March. Amidst all this, there's a persistent urge to do more, whether it's getting into open-source initiatives or sharpening coding skills. Each week brings new revelations and challenges, moving me further down the path of becoming a BitDev.

### HD Wallets & Extended Private Keys

This week, our focus shifted to Hierarchical Deterministic Wallets (HD Wallets), a significant advancement in Bitcoin wallet technology. Serving as a second-generation solution, HD wallets offer substantial improvements over their predecessors, addressing crucial aspects such as address generation, key management, and transaction simplification. The cornerstone of HD wallets is the Seed, which serves as the foundation for generating every address. Analogous to a tree's solid trunk, the Seed gives rise to each private/public key pair, ensuring seamless and secure Bitcoin transactions. Now getting into the mechanics, we explored the derivation of the Master Private Key (m) from the Seed, followed by extended private keys (xpriv), and the crucial role of the chaincode in key generation.

### Extended Public Keys

Extended Public Keys (xpubs) are instrumental in facilitating Bitcoin transactions, allowing for the generation of corresponding public keys essential for verifying transactions. Derived from the Seed through more cryptographic processes, xpubs do not grant direct access to funds but serve as a tool for generating public keys and verifying transactions. Utilizing a hierarchical structure ingrained in HD wallets, each xpub corresponds to a specific branch of the wallet hierarchy, enabling efficient and secure transaction processing.

### Hardening Keys: Adding an Extra Layer of Security

Hardened Extended Private Keys (xpriv') and Extended Public Keys (xpub') enhance the security of HD wallets through a process known as “hardening.” Marked with a ' symbol, hardened keys ensure that their corresponding child keys cannot be computed without the original key itself, protecting funds and ensuring transaction integrity. This additional layer of security is crucial for secure Bitcoin wallet management.

### Backups

Fortunately, when it comes to backing up our seed, we're not stuck memorizing a random string. Converting a seed into a set of 12 or 24 mnemonics is like simplifying complex code into an easy-to-remember passphrase. Instead of dealing with convoluted strings of characters, you're presented with a sequence of straightforward words. This mnemonic phrase acts as a backup for your wallet, making it incredibly convenient to jot down or commit to memory. It also provides an additional layer of security. Even if you misplace your original Seed, you can effortlessly regenerate it using these mnemonics (as long as you keep them in a safe place). Think of it as having a secret key to unlock your treasure chest, ensuring the safety of your bitcoin.

### Exploring Code and Rust-Bitcoin Projects

After encountering setbacks last week and struggling to find a suitable project, I decided to return to the fundamentals of Rust. To level up my skills, I dove into Rust practice problems offered by open-source projects, with Rustlings being a favorite of mine. Rustlings is an open-source project designed to help learners master Rust. It offers a series of progressively challenging exercises that cover various facets of Rust syntax, semantics, and best practices. Afterward, I took on the task of exploring numerous Rust-focused, bitcoin-related open-source projects, likely sifting through more than a hundred of them. Eventually, I whittled down my options to nine projects that truly piqued my interest and resonated with me, promising both fulfillment and excitement in my work.

I decided to look into these projects because after soloing projects for so long, following online resources or tutorials, I realized this approach is time-consuming and inefficient. I came to understand the value of feedback, whether positive or negative. I also recognized the necessity of working on tangible, real-world applications — specifically, those related to bitcoin. It's time to stop dodging the punches and start coding for bitcoin applications directly. These are the projects and software that I'll be working on once I secure an internship with Summer of Bitcoin. My aim is to gain practical experience and learn from any mistakes I encounter.

The projects I found extremely cool were: Fedimint, Lightning Development Kit (LDK), and Bitmask. All of theses projects are focused on second-layer solutions built on top of Bitcoin, and they all seem to have welcoming developer communities. So, today marks my first step as I join one of the Fedimint Dev Calls, hoping to meet some of the team members and find a good first issue. Let's get it.

### In Summary

My third week in the Summer of Bitcoin has been truly remarkable. There's nothing I'd rather spend my time on than learning and building within this space. Through hands-on sessions and project exploration, I've gained valuable insights into the intricate mechanisms of bitcoin. I have a newfound appreciation for security measures and practical applications in bitcoin and, I eagerly anticipate contributing to impactful projects like Fedimint, LDK, or Bitmask (maybe all). I'll continue navigating these challenges and seizing opportunities until I become a BitDev so, let's get after it.

Thank you for joining me and If you found this article insightful or have any questions or critiques, I'd love to hear from you. Feel free to reach out to me on Twitter or LinkedIn.
