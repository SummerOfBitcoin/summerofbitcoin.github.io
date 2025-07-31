---
layout: post
title: "From Bitcoin Newbie to BitcoinJS Contributor"
date: 2024-07-15
author: Ayman Mohammed
categories: [BitcoinJS, Stories]
image: https://cdn.hashnode.com/res/hashnode/image/upload/v1724825570742/2790eb1a-8f7b-4c15-bc4f-3896c91f9935.png
---

## Introduction

I'm Ayman Mohammed, a CS grad at Hyderabad. I got into the crypto space around this time last year when I joined a startup as an intern in my final year of university, which primarily focuses at bringing Bitcoin to DeFi. The main product was a trustless bridge that uses HTLCs to swap Bitcoin to Wrapped Bitcoin on EVM chains and vice versa.

I got to know about Summer of Bitcoin initially by [my friend Jayendra](https://jayendra.hashnode.dev/) who was also a fellow intern at the same startup. We applied together for Summer of Bitcoin and both of us eventually got into the first round.

## Round 1: The selection process

The selection process involved reading chapters from Grokking Bitcoin each week. The round lasted for 4 weeks and at the end of each week we joined a zoom meeting with about ~200 (?) participants. Everyone was broken up into smaller breakout rooms.

A single person is chosen as the host to move the meeting forward in case of conflicts, or uncooperating participants. Each breakout room had 5-6 participants where we asked questions to each other to solidify our understanding on the assigned reading that was to be done that week.

This round was essential to solidify my understanding on Bitcoin. Prior to this, I only worked on EVM, and had little knowledge about the Bitcoin ecosystem. Some of the topics this book covered are:

* Basics Elliptic curve cryptography
    
* Mnemonic phrases
    
* Bip32 Derivation paths
    
* Mining consensus
    
* p2pk, p2pkh, p2sh payment types
    
* Bitcoin Transactions
    
* Proof of work consensus
    
* The p2p network
    
* segwit
    
* Bitcoin hardforks and softforks
    
    and much more!
    

The round required a participation of 80%. At the end of each zoom meeting there was an attendance form that was to be filled. After 4 weeks of studying Bitcoin, me and my friend moved forward to round 2.

## Round 2: Mini Bitcoin Miner challenge

Yep, the assignment was to really build a bitcoin miner. I was stumped when I first came across this without knowing what to do. I had no plan I jumped headfirst into building the script interpreter only to get to know that it wasn't even required later 🥲.

I took a step back and really thought about what it would take to validate a Bitcoin Transaction. I started of by validating script types, checking addresses (with base58, bech32, bech32m encodings), double spending, locktime / nSequence checks.

Validating p2pk, p2pkh, p2wpkh was required at the very least. Apart from the above checks this would also require validating signatures. There were no p2pk transactions in the mempool given to us so I skipped that.

Once I had my signature validation ready, I started working on making the coinbase transaction. I included this with other transactions in my block. The other transactions were selected the block in a greedy fashion by taking the fee / weight ratio and sorting them in descending order. After generating the block, I pushed my code. There were a few bugs, but the initial implementation passed! [Here is the link to that commit.](https://github.com/SummerOfBitcoin/code-challenge-2024-Nesopie/tree/d12dea0011b3336f3b141a60f3b342868ee3b247)

Once the base was set, I moved forward with more harder tasks which were probably not required for the assignment. I validated p2sh and p2wsh and nested segwit transactions. For this I also had to create a script interpreter. I will talk about the script interpreter and my implementation in another post but you can [browse this folder to get a rough idea.](https://github.com/SummerOfBitcoin/code-challenge-2024-Nesopie/tree/main/src/features/script) I've only implemented the opcodes which were in the script-based transactions inside my script interpreter so it is not yet complete.

I also implemented Taproot transaction validation (only for key path spends). As for script path spends apart from control block validation, I did not check for script execution.

I implemented transaction caching (for sighashes) as well, and "despite writing my project" in typescript and implementing way too many features that were not required for the project, my implementation ran for the shortest amount of time and had the highest score (to my knowledge).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724825570742/2790eb1a-8f7b-4c15-bc4f-3896c91f9935.png align="center")

I mined more fees than the fees required by the assignment so I ended up getting a score higher than 100 (my score was 112). My code also usually runs for ~30 seconds to validate 3k transactions. Although there were 11k (?) I mined only 3k of them in my block. This is because of my greedy selection criterion.

After running completing the assignment you realise that all the transactions in the mempool were valid (they were scraped from a foundry block LOL) :).

## Round 3: Selecting my organizations

I was excited to see that BitcoinJS was one of the organizations that were coming up this year. I came across BitcoinJS when I was writing my assignment. Using their transaction class was helpful to find the fees for transaction. It was also through BitcoinJS when I realised that every transaction is valid. (If they were invalid then the Transaction class would through an error).

For me, language wasn't a barrier since I've dabbled in a lot of languages. Another organization I came across was CoinSwap. CoinSwap aimed to achieve privacy on Bitcoin by (as the name suggests) swapping your UTXOs. This was done through a maker and taker p2p network. It used HTLCs and the entire network formed a ring where you keep swapping utxos with them until they eventually come back to you. Since I've worked with HTLCs and Rust before, I also put this organization in my watchlist.

I applied to BitcoinJS and CoinSwap. More specifically, 2 projects in BitcoinJS and 1 project in CoinSwap. These projects were:

* Adding BIP370 support (PSBT v2) to the BIP174 repo.
    
* Add hybrid module support (ESM and CJS) to the BitcoinJS ecosystem.
    
* Use the BDK wallet API to replace in-house logic in CoinSwap.
    

Two weeks after applying to the organization, I got a mail saying that I got into BitcoinJS :).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724827466481/ee131093-4a97-4a9f-9e1d-181b740c5c5f.png align="center")

## Conclusion

I'm really glad that the Summer of Bitcoin program is an actual thing. This program has a direct impact on what I'll be working on in my career and I'm glad it did. :)
