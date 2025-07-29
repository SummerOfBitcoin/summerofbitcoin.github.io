---
layout: post
title: "How I Cracked Google Summer of Code and Summer of Bitcoin"
date: 2025-06-15
author: Krrish Sehgal
categories: [Caravan, Stories]
---

## The GSoC Discovery

I first heard about GSoC through [Apna
College](https://www.youtube.com/@ApnaCollegeOfficial). By January 2024, I was
_trying_ to explore it — but with **zero serious dev experience**. Sure, I
knew basic HTML/CSS/JS, but that alone won’t get you far in open source.

Instead of applying half-heartedly, I made a decision: **focus on building and
learning first**.

Even though I didn’t apply in 2024, seeing LinkedIn posts of first-year
students getting in made me aware of what’s possible — but I knew I had made
the right call for myself.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*9tbI88NZPc24VDt6hlPQPA.jpeg"/>
</figure>

## Setting Goals & Getting Obsessed

Fast forward to October 2024: during my second internship, I hit a moment of
personal reflection (I chickened out of talking to someone I wanted to), and
that triggered a weird but effective motivation. I told myself, “If I can’t do
that, at least let me start working towards something meaningful.”

So I got up, sat down, and started looking for GSoC organizations. I found one
with a Django codebase — something I knew nothing about — but also it had a
semi-active Flutter project.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*G3An9fRjDOaHQRfc2L0q1A.png"/>
</figure>

[My first ever merged contribution](https://github.com/OWASP-BLT/BLT-
Flutter/pull/461)? A Flutter feature related to copy-paste functionality. I
learned how **Flutter communicates with native Kotlin using method channels**.
That one merge gave me the confidence to keep going.

A couple of weeks in, my GSoC org released a
[leaderboard](https://github.com/OWASP-BLT/BLT/issues/2916). It showed top
contributors, and I went back to previous year leaderboard and noticed someone
who cracked GSoC 2024 had **50–60 PRs** merged. That’s when I set a mental
goal:

> I’ll get at least 40–50 PRs in before May.

That’s when the obsession kicked in.

Each PR felt like a kill in Fortnite or a perfect Wu Shang combo in
Brawlhalla. From breaking my head over failing test cases to finally seeing
that purple merge button — **the dopamine hits were real**.

## Taking on Tougher Issues

One major turning point was when I worked on a [**call-blocking
extension**](https://github.com/OWASP-BLT/BLT-Flutter/pull/464) in Flutter for
the OWASP BLT app. It needed deep native integration — something Flutter alone
couldn’t handle. I had to dive into Swift for the [iOS
side](https://github.com/OWASP-BLT/BLT-Flutter/pull/465). It took 10–11 hours
daily for a week.

Another feature was a **plagiarism detection mode** , [integrating
OpenAI](https://github.com/OWASP-BLT/BLT/pull/3160). I even [started with
Hugging Face](https://github.com/OWASP-BLT/BLT/pull/3134) but later pivoted
due to size issues. This forced me to learn about **vector comparisons** and
**algorithmic matching** — a super technical(concepts like abstract syntax
trees, Levenshtein distance or difflib) and satisfying experience.

When I’d get stuck, I’d remind myself:

> “If I don’t solve this one specific problem, I’m not going to make it — so
> stop whining and fix it.”

# The Bitcoin Chapter: Ordinals, Runes, and Mainnet Nodes

In January, I pushed myself to pick a challenge that no one else had solved in
my GSoC org: **deploying a token on the Bitcoin blockchain using the new
Ordinals + Runes protocol** (released April 2024).

It was one of the toughest things I’ve ever done. I ran four servers synced to
the Bitcoin mainnet, one on regtest, and another running an Ordinals server. I
dove deep into how [Ordinals](https://github.com/ordinals/ord) and Bitcoin
internals work, even reaching out to core maintainers like Greg, ETS, and Json
on the Ordicord Discord server to get everything up and running.

Eventually, on **Feb 21, 2025 @ 22:40:11 UTC** , I successfully etched my
token. That experience? Mind-blowing. [_(Link to token proof
here)_](https://ordinals.com/inscription/9ebf269d5b73ade615e439cbb0ed6427697672e7c60b761dc2e566f4a5a80050i0)

This deep dive sparked my interest in Bitcoin. And just when I thought I’d
take a breather, I saw a LinkedIn post:

> “Summer of Bitcoin applications are open!”

# The Summer of Bitcoin Journey

I applied for SoB the very next day. The entry process involved solving **3
technical challenges** , each with a 1-week deadline.

The tasks were amazing — they taught me about:

  * How Bitcoin transactions are formed
  * SegWit
  * Multisig wallets
  * And more low-level transaction logic

Since I already had experience deploying a Bitcoin node for my GSoC org
project, **Challenge 1 (setting up regtest)** was a breeze.

I completed all three in two weeks and was selected for the next round. They
were the hardest two weeks up until then because it involved a lot of learning
on the go about the bitcoin blockchain.

# Picking the Right Org for SoB in the Proposal Round

By March, my GSoC selection felt locked in. But I had _no motivation_ left for
SoB. Life threw in some perspective again, and I realized if I didn’t start
contributing, I wouldn’t have a good proposal.

## Finding Unchained-Caravan

I started again with only 3 weeks left for proposal submission deadline and
explored all the listed SoB orgs and shortlisted based on two things:

  1. Number of projects under each org
  2. Complexity and fit for me as a dev

Caravan had 3 projects and relatively **less competition**. It was a
**monorepo with both packages and a website** , which was ideal. I began
contributing to the website, then gradually shifted to the core package.

But none of my early PRs were eye-catching. So I made a **bold move** :

> I started building the entire proposed project myself.

Within a week, I pushed 2 working but unrefined
[PRs](https://github.com/caravan-bitcoin/caravan/pull/246) that covered 4**0%
of the one of the project proposal** listed under Caravan.

The code wasn’t pretty — but it worked. And that’s what open source is about.
Someone could’ve even used my code in their proposal — and I’d have been
totally fine with it. That’s the beauty of community-driven building.

# The Final Results

**May 5, 8:30 PM:** I was at the gym, just about to hit a leg press set. I
checked my email —

> I got selected for Summer of Bitcoin.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*I1YXLY0gyW8OBTKnNo1Vaw.jpeg"/>
</figure>

I was so pumped I had the best workout of my life.

**May 8, 11:35 PM:** I got the GSoC result. I was already confident by
February when my token was etched. Seeing that final confirmation was just the
cherry on top.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*_227_Ejgx7VP2PRo-TIKOw.png"/>
</figure>

# How I Searched and Selected Organizations (GSoC and SoB)

I went full detective mode.

For GSoC, I connected with people on LinkedIn and just **kept asking
questions** — how they chose their orgs, what tech stacks they used, how they
contributed. Then I explored those orgs myself and often landed on different
projects within the same org.

For Summer of Bitcoin, it was even more intense. I had much less time, so I
relied heavily on willpower. I listed all the organizations and chose the one
I genuinely enjoyed working with — one that had a tech stack I was somewhat
familiar with, an active community, and relatively less competition. I had to
learn Bitcoin concepts from scratch and quickly build something meaningful.

Some tips:

  * Use [gsocorganisations.dev ](https://www.gsocorganizations.dev/)to explore orgs based on tech stack, difficulty, and history.

# How to Write a Strong Proposal (Especially for SoB)

One word: **Contributions**.

Your proposal is secondary — **your code speaks louder**. That’s what worked
for me in both cases.

Sure, I’ve seen people get selected with 4–5 contributions and killer
proposals — but not in the orgs I was in. They all expected meaningful code
contributions _first_.

So if you’re spending hours tweaking the formatting of your proposal and
haven’t merged a PR yet — **rethink your priorities**.

