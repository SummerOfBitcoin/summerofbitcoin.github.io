---
layout: post
title: "Bitcoin Light Clients Explained: A Beginner's Guide to Neutrino"
date: 2023-07-23
author: Maureen Ononiwu
categories: ['Tutorials']
image: https://www.tutorialspoint.com/blockchain/images/chaining_blocks.jpg
---

## Summary of What to Expect From This Article

This article begins by giving the reader a brief background on Bitcoin. This background is aimed to help you understand the need for neutrino. Going further, I also gave a brief overview of how neutrino works.

## A Background On Bitcoin.

Bitcoin can be described as a new kind of money that eliminates central authorities or banks. To better understand this let us draw a contrast between this type of money and what is currently in vogue, the fiat system i.e. the naira, dollars, and euro that we are more familiar with. In Nigeria, the CBN Act Section 2 (b) mandates the Central Bank of Nigeria to be the sole issuer of legal tender currency in Nigeria. In addition to this, one also has to depend on financial institutions to verify and manage transactions as they solely have the transaction data required to carry out this process. In these two instances, you can draw a conclusion about the fiat system, *compulsory trust, and dependence on a third party to drive the monetary system*.

Bitcoin eliminates this by leveraging what is known as a peer-to-peer technology, blockchain. The idea is that each and every participant in the system has their own copy of the transaction data and so can independently verify transactions as well as participate in a process known as mining to issue new bitcoin.

## The Need For Light Clients Like Neutrino

Although the elimination of central authority is one of the principles that have guided Bitcoin's development, we have seen some tendencies toward centralization in the Bitcoin system. One of the reasons why everyone in the Bitcoin system do not run a node but rather depend on a third-party node is the amount of computing resource required to run one. This has given rise to the need for light clients. Light clients in this case are bitcoin clients that can verify transactions without needing as much computing resource as the regular bitcoin client.

There are two Bitcoin light client implementations specified by [Bip-0157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki) and [Bip-0037](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki). Bip-0037 is an older implementation of the light client that makes use of bloom filtering, over time it has been criticized as giving wallets zero privacy and being susceptible to falling prey to malicious full nodes. The Bip-057 (The implementation that birthed neutrino) comes as an improvement to the lightning client protocol

## Overview of how neutrino works.

According to the Neutrino GitHub repository, Neutrino is a Bitcoin light client written in Go and designed with mobile Lightning Network clients in mind.

To understand how it works, we would have to first understand, the structure of blocks in the blockchain.

![Chaining Blocks](https://www.tutorialspoint.com/blockchain/images/chaining_blocks.jpg align="left")

(Image gotten from [tutorialspoint.com](https://www.tutorialspoint.com/blockchain/blockchain_chaining_blocks.htm))

The diagram above is a pictorial representation of the structure in which data is stored in Bitcoin, hence, the name blockchain. "Blocks" are linked together such that the next block contains the hash of the previous block. Each block consists of a header and transaction data. The regular Bitcoin client i.e. the full node client downloads the entire block data while light clients are designed to download only block headers. Neutrino verifies the header it gets from its peers by ensuring the previous hash of a block is the same as the hash of the preceding block. It also verifies header's proof of work and target difficulty as well as makes use of "filters" to fetch transactions or blocks it is interested in from full-node peers to maintain privacy.

## Summary

In this article, I did a refresher on Bitcoin and helped the reader understand the need for light clients such as Neutrino as well as an overview of how it works. In the next series of this blog, I would be diving deeper into the technicalities of how neutrino works.

**Reference**

1. [https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki)
    
2. [https://www.tutorialspoint.com/blockchain/blockchain\_chaining\_blocks.htm](https://www.tutorialspoint.com/blockchain/blockchain_chaining_blocks.htm)
    
3. [https://www.bitstamp.net/learn/crypto-101/what-are-blocks-in-the-blockchain/](https://www.bitstamp.net/learn/crypto-101/what-are-blocks-in-the-blockchain/)
    
4. [https://en.bitcoin.it/wiki/BIP\_0037](https://en.bitcoin.it/wiki/BIP_0037)
    
5. [https://www.cbn.gov.ng/currency/legaltender.asp#:~:text=The%20CBN%20Act%20Section%202,ensure%20monetary%20and%20price%20stability](https://www.cbn.gov.ng/currency/legaltender.asp#:~:text=The%20CBN%20Act%20Section%202,ensure%20monetary%20and%20price%20stability).
    
6. [https://github.com/lightninglabs/neutrino](https://github.com/lightninglabs/neutrino)
