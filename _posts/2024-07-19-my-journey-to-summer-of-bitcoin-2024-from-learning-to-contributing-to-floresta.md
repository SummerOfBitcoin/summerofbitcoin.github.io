---
layout: post
title: "My Journey to Summer of Bitcoin 2024: From Learning to Contributing to Floresta"
date: 2024-07-19
author: "Abhinav Agarwalla"
categories: [Stories]
---

### What is SoB ?

Summer of Bitcoin is a global program focused on bringing more student developers into the open source software development in the Bitcoin community. Students work with an open-source organisation on a 12 week programming project during their summer break from school.

This blog highlights journey of getting selected into the program from the very beginning. I believe their is no defined path for getting selected in such open-source fellowships. Everyone has a unique journey, and you can take a few things from other journeys to write yours :) because the essence of each one is similar.

### How I came to know about SoB ?

The first time I heard about SoB was in an event conducted by my college seniors where I was introduced to open-source programs like GSoC, LFX etc. and people who were previously selected shared their experience with us.

### Learning Phase

It was a 2 month long selection period.

**For the first month:**

We were assigned with learning and understand the core technicals of the Bitcoin protocol and how different operation in the network are carried out inn theory.

For this we were recommended to read the Grokking Bitcoin written by Kalle Rosenbaum, 2 chapters a week for the next 4 weeks, and every week joined an online break room session of 6 people in which we would ask each other questions about this week's reading and talk about Bitcoin.

While reading the book, I felt the need of some hand's on coding experience on blockchain related projects, to understand what I am reading in a better way. So I started exploring the internet for some blockchain programming related tutorials.

I had some experience writing in Rust from some previous projects and found a tutorial in which guy built a simplified version of a blockchain system for learning purposes. So I started following this tutorial along with reading Grokking Bitcoin for the next month.

**For the second month:**

We were assigned with a coding challenge, to test our knowledge about Bitcoin.

The assignment was to successfully mine a block while correctly validating the transactions according to the Bitcoin consensus rules, and twist was we could not use any bitcoin verification related libraries.

So after a few days of brainstorming, research and asking around in the community, it was getting clear on how to start coding.

For learning about the components of different bitcoin transactions LearnMeABitcoin was a pretty great resource, but sadly I didn't heard about it unit the very end, so I had to take the long road and scour through the internet learning about different kinds of transactions and how to verify their signatures, I read Bitcoin developer Docs and BIPS to get a clear picture. BIP 143 helped me the most to figure out the segwit transactions.

So I coded for around 15 days, day and night, solving one bug after the other, implementing one verification logic after another.

So during the coding days, I almost quit about 2 times. This assignment was one of the hardest to be assigned in all the previous years. After going through everything, completed the assignment.

LOL, you must be confused about the 101. Actually our score was calculated on the basis how efficiently we selected the transactions, carefully optimising the logic to prioritize those transactions with less weight units and high gas fees. As a result I crossed the maximum expected gas fees in the block, so my score also crossed 100.

You can take a close look at the assignment here.

### Selecting the Org I wanted to work with

After the coding challenge, my proficiency on Rust increased a lot. So I made up my mind to contribute in a project based on Rust.

Rust is getting widely adopted in the Bitcoin community because of its high performance and memory safety. So while researching about different orgs, I came across FLORESTA.

Floresta is a full node implementation written in Rust by its lead maintainer and my mentor Davidson Souza , which uses Utreexo proofs for UTXO verification and storage, with an integrated Electrum server, for Electrum clients. Using Utreexo greatly enhances the huge storage crisis faced by the full nodes, as now we do not have to store the whole UTXO set in raw form locally but only in the form of cryptographic hashes. This greatly reduces the storage needed for the blockchain state.

The project I went for was to create a robust testing environment for Floresta. This included increasing unit test coverage, checking the overall performance for node through integration testing, load testing which measures the endurance of the software.

I started checking out its Contributing Guidelines, resources to get started, its codebase and made a few contribution, enhancing the error handling in some files.

### Writing Proposal

It is expected for proposal to contain a detailed explanation of your implementation and timeline. What gives you an edge is your contributions to the project until then. This helps solidify the trust in your abilities to finish the project.

I got up to speed on different features that Floresta provides, and what kinds of integration tests I can simulate. While researching for ideas, I also came across some unique kinds of testing, like :

*   **Load Testing:** By simulating a large number of clients connecting to the server and request data at the same time. JMeter is a tool used for this.
*   **Scalability Testing:** Limit the CPU, RAM available to the system to see how it affects the performance metrics.
*   **Fuzz Testing**
*   **Regression Testing**

I made sure to add code snippets and mock implementation of how the code will be implemented in Rust. It is also a good idea to add in diagrams which I didn't, because images speak more than plain text.

My mentor seemed happy with the design because they didn't seem to have much issue with what I had shown them in draft.

But Lakshya Singh , one of my seniors I requested to review my proposal, told me that it was less technical and need more of it. This pushed me to improve my proposal further, add more details about the implementation, add in more code snippets.

### Post Proposal Submission

Going inactive after proposal submission shoes that you aren't interested in the community, and was just doing it for the namesake.

Fast forwarding to 7th May, the day when our results were to be declared. But during that time my mentor was busy in some events and also Bitcoin Asia was also going on, so my org's results were delayed about 10 days. It was a very stressfull few days, as all my friends were getting their selection emails and I had no idea of my fate.

Finally on 17th May, one of my friends from IIT Kanpur, with whom started taking during the assignment period on Discord, we used to help each other with our doubts. One more amazing benefit you get form engaging in open-source communities, is you get to meet some amazingly smart people on the way.

So it was around 5 in the morning, YES before even dawn :), he texted me that my org had released their results, as it was visible "Announced" in the SoB website. I was literally sleeping and just scrolled my emails, half asleep, and saw "Congratulations" and then literally shouted woohoo!

### Final thoughts

SoB is a non-competitive program, so try to cherish the community alongside it. Find your motivation, why do you even want to do it first, and then rest will follow.

At the end, everything is achievable through consistent hard-work alone, you become talented on the way.