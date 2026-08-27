---
layout: post
title: "Eliminate backdating in the replacements of Replace-by-fee in Bitcoin Core."
date: 2026-08-17
author: Biel  # Must match the name in _config.yml
categories: Bitcoin-Core  # See available categories below
image: ../assets/images/blog_content/2026-08-17-bicaru20-avatar.jpg
---

## Who I am and what I worked on

I'm Biel, a student at UAB. This is my first time with summe of Bitcoin and I have been working on Bitcoin Core. My project aimed to eliminate the backdating in the replacements of Replace-by-fee. This behaviour harms the users privacy as it gives away that the transaction was build with the Core wallet.

## Main objectives

For this summer, there were two main objectives:

The main objective was to fix the current behaviour of the Bitcoin Core wallet so that transaction locktime is not backdated when performing RBF. Currently, in Bitcoin Core, given an original transaction A and its replacement B, we refer to backdating when the locktime of A is higher than the locktime of B (`A.locktime > B.locktime`). This can happen because Bitcoin Core enables anti-fee-sniping by default, which sets the transaction's locktime to the current block height. For privacy, 10% of the time, it instead sets a different locktime, randomly chosen between the current height and the current height minus 100 blocks. This can lead into having a replacement of a transaction with a locktime older than its original transaction. This is unrealistics and can be used as a wallet fingerprint. You can find a test that reproduce this problem [here](https://gist.github.com/Bicaru20/9afcb878dd103369255cd81cffac86ba).

Also, at the same time we wanted to perform a historical analysis of locktime behaviour across the Bitcoin blockchain. This would allow to undertand the behaviour of the locktime and at the same time help us decide what approach is best to solve the backdating problem.

## What I have been working on 

I have been working on [fixing the locktime](https://github.com/Bicaru20/bitcoin/pull/3) backdating behaviour in Bitcoin Core when replacing a transaction through `bumpfee`. The proposed solution keeps track of the previous locktime through `CoinControl` and passes it to `DiscourageFeeSniping` function through a new parameter. With this fix, the replacement of a transaction cannot have a locktime older than its original transaction even if it backdates. The corresponding functional test was also added to `wallet_bumpfee.py` to verify that the replacement transaction is not backdated.

During the implementation of this fix, a second issue was identified. We found that when replacing a transaction, if the original transaction had a time-based locktime, the replacement would still apply the anti-fee-sniping logic. This could cause the replacement to change its locktime type, making the transaction highly fingerprintable as originating from Bitcoin Core. The solution was simply to check if the original transaction had a time-based locktime. This is the [pr](https://github.com/Bicaru20/bitcoin/pull/4) with the fix. And [here](https://gist.github.com/Bicaru20/e86b46a81b89527ed0c8b4ce377e827d) a test to reproduce the issue.

In parallel, using rust and python, we gathered historical locktime data across the entire blockchain. We decided not to publish a preliminary analysis yet, as we want to conduct a more rigorous analysis and publish the results as a scientific paper.

## Future work

The next stage is to complete the review and integration of both PRs. We will also use the collected data to perform a more rigorous historical analysis of locktime behaviour, with the goal of developing the results into a rigorous analysis.

## What I learned

I learned a lot about the Bitcoin Core codebase. Even though my PR does not modify a large amount of code, I had to investigate and work with several different parts of the Core codebase to understand the behaviour and implement the fix.

At the beginning, I was intimidated by the idea of working on a project with so many lines of code. However, this project taught me that Bitcoin Core is not as overwhelming as it may initially seem and that it is actually very manageable once you understand the relevant parts of the codebase.

## What's next

When both prs are merged or rejected, I will keep working on Core by doing mostly reviews for now. Also, other analysis about other aspects of the Bitcoin blockchain will be conducted to undertand better the behvaiour of the blockchain.