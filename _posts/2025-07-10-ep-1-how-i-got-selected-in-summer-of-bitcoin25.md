---
layout: post
title: "Ep 1: How I got selected in Summer of Bitcoin'25"
date: 2025-07-10
author: Abhishek Raj
categories: [Caravan, Stories]
---

**Hey everyone,**

I’m **Abhishek Raj**, a third-year student at **NIT Durgapur** and yes, I got selected for **Summer of Bitcoin 2025**.  
Also, I know I’m *super* late in writing this blog. Still, I really wanted to sit down and share the full story.

So, let’s rewind the tape to where it all began...

### How it all started ?

It all started in my second year of college when I was casually scrolling LinkedIn (as one does), and I saw a post from my college senior about SoB. Out of curiosity and a bit of FOMO I went through the website, and boom, I started reading about Bitcoin, and honestly, it felts so fascinating at start as it all completely new for me . Until then, I only knew Bitcoin from movies where hackers used it to stay anonymous 😅

I started wondering: *What exactly is decentralized money? Why does fiat feel like Monopoly cash in comparison? How the heck does all this actually work? Why do we even need bitcoin ? S*o, I started reading about decentralized currency, how it’s different from the regular money we use, and how open source plays such a big role in making it all work. The more I explored, the more it fascinated me: a system that works without banks, powered by people and code. Even though I knew the selection rate was low and it wouldn’t be easy, I decided to just go for it. Worst case, I’d learn a lot and best case?  
Well, here we are.

## **The Assignment Phase: Where the Real Fun Began**

Ah yes, the assignment phase, this is where the actual fun (and chaos) begins. Honestly, this part is what really sets **Summer of Bitcoin** apart from other programs. We were given four technical assignments, each meant to be completed week by week, but all of them were unlocked from the start. That meant you could pace yourself, binge them like a Netflix series or suffer one episode at a time. 😅 For me, this phase was one of the most intense and rewarding parts of the whole journey. I was learning new things every single day and actually applying them.

Week 1 and 2 were pretty smooth, I was vibing, learning, and building with excitement. But then came **week 3**, and oh boy, that was a whole different beast. It was hands down the toughest part. Everything just hit harder constructing block headers, computing merkle roots, raw hex structures it felt like my brain needed a hard reset for real, that It took me over a week to get through just that one task. But weirdly enough, I loved every second of that struggle. It pushed me to understand how things really work under the hood, not just how to make something run. And by the end of it, everything made way more sense. Looking back, this phase taught me how to learn fast, debug hard, and keep going even when I felt stuck.

So, what I was doing that I opened up different tabs, read everything possibly available on the internet, also used ai too (like why not) understand everything, asking 100s of doubts with different examples, even discussing different approaches. During this time SoB discord community helps me lot too. So, after clearing the screening round, all shortlisted candidates were added to a dedicated Discord server, and honestly, it turned out to be one of the best online spaces I’ve been part of. No spam, no ego, no useless noise just pure collaboration and knowledge-sharing. People were constantly discussing ideas, helping each other debug tricky problems, and sharing different approaches. It wasn’t just helpful it was motivating. I made sure to be part of that energy too, trying to help out others wherever I could.

Two resources helped me a lot during this phase: the book ***Grokking Bitcoin*** (you can find the PDF online **\*must read book) and the website** [**Learn Me a Bitcoin**](https://learnmeabitcoin.com)**.**

## The Proposal Arc: Picking the right one

After completing all the assignments, it was time to pick a project and write the proposal. I didn’t want to just randomly choose something, I wanted to work on something I genuinely understood and felt excited about. That way, I could write a stronger, more detailed proposal *and* start contributing early, which I knew would increase my chances of getting selected. I wanted to show the mentor that I wasn’t just interested, I was ready to hit the ground running.

That’s when I found **Caravan**, a fully open-source, stateless multisig coordinator. What instantly caught my attention was the tagline on the website:

> **\"Caravan simplifies multisig custody by coordinating your transactions without storing your keys. Self-custody your bitcoin with enhanced security, privacy, and protection against single points of failure.\"**

That line *clicked*. The idea of improving the testing framework for something so critical, something that handles **wallet creation, transaction signing, and private key management** felt so fascinating. One small bug here could break the whole experience or worse, compromise user safety. I also realized how testing could play a major role like for eg, in making sure sensitive data like **xpubs** remain protected during dev workflows. So, I dove into the codebase, explored the test coverage, and started figuring out how I could actually help not just by fixing things, but by improving the entire setup.

Here are a few key ideas from my proposal:

* **Implement E2E tests using Playwright** to automate the full multisig flow from importing wallets to signing and broadcasting transactions against a Bitcoin Regtest network.
    
* **Upgrade Caravan’s CI** to include Codecov for tracking test coverage and run both Vitest & Playwright on every PR.
    
* **Add extra test cases** for edge cases like malformed PSBTs to make sure Caravan doesn’t crash on invalid input.
    
* You can read the full proposal here:
    
    [https://docs.google.com/document/d/1G7KCWU7pAlH00v6ovXaeED\_6MOQ2N5IehlYy65Mtm74/edit?usp=sharing](https://docs.google.com/document/d/1G7KCWU7pAlH00v6ovXaeED_6MOQ2N5IehlYy65Mtm74/edit?usp=sharing)
    

## ⏳ The Waiting Game… and the Good News

The proposal submission window started from **March 16**, but here’s the twist new orgs were still being added till around **March 28**, so I was kind of in wait-and-watch mode. At first, I was choosing between like **Nostr**, **Braidpool**, and **Breez** and honestly, I spent way too much time thinking about which one to go for. I finally locked in on **Caravan** on **April 3**, because the idea just clicked with me.

Oh, and I totally forgot to mention, all of this was happening *right in the middle of my end semester exams*. 😅 The proposal submission window was from **March 16 to April 25**, and my exams were starting from **April 20**. That meant I had just approx **15 proper days** for the first draft, and to give my full focus to the proposal before I’d be buried in exam prep. And trust me, I didn’t want to look back later and think, “Bro, just because of one exam, I missed this opportunity.” So I gave it my all. Wrote, rewrote, reviewed everything and finally hit submit on **April 25**.

After that, I went straight into exams. My last paper was on **April 29**, and once that was done, the real waiting game began. Every morning from May 1st, I was checking my inbox like a maniac although its already mentioned on the website that result will be announced on 5th may but still.😭 And then, on **May 5** mark this date guys **THE mail came**. The subject line said:

> **Congratulations, you’ve been selected!**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1751563082500/71988906-54e2-457f-b750-6d0552c5ba06.png align=\"center\")

I just froze for a second. Read it again. And then shouted like crazy. That one moment made everything worth it the sleepless nights, the assignments, the failed test runs, the nervous proposal editing *all of it*. I still remember sitting in my room, just smiling like an idiot, thinking, “I actually did it.”

It felt unreal.

### What’s Next?

Now that I’m officially in, the real work begins. I’ll be

* Migrating **Caravan’s test suite from Jest to Vitest** for better performance and modern tooling.
    
* Writing **Playwright tests** to simulate real user flows like creating wallets, signing transactions, and more.
    
* Improving the **CI pipeline** with test coverage tracking, code formatting checks, and safety features to catch bugs early.
    

I’ll also be documenting my journey through more blog posts sharing my progress, the problems I face, the things I learn (sometimes the hard way), and maybe even a few funny bugs along the way. 😄

One last thing I want to say, this post isn’t about showing off. Getting selected for Summer of Bitcoin definitely gives you some amazing perks, like mentorship and connections, but you *don’t* need SoB to get started in the Bitcoin ecosystem. Open source is open to everyone. You can pick a project, fix a bug, ask questions, and start contributing. That’s the beauty of it you learn by doing, and you grow with the community.

Thank you so much for reading this far. See you in the next one!
