---
layout: post
title: "Utreexo: A Deep Dive into Improving Bitcoin's Scalability"
date: 2024-08-17
author: "Njokom Nshanui"
categories: [Tutorials, Utreexo]
---

The world is growing ever digital by the day and more and more aspects of our lives have become more and more intangible. Currency hasn't been left out, from physical cash, to cashless systems and now cryptocurrencies. Since the inception of the first block of Bitcoin in 2009 it's adoption has grown ever so rapidly and with more adoption comes more stability and trust in the network which will only increase it's adoption.

First of all, what is Bitcoin, and how does Bitcoin work?

Bitcoin is a decentralized peer to peer network digital currency created in 2008 by a pseudo anonymous entity named Satoshi Nakamoto. The Bitcoin network is a network made up of computers, also referred to as nodes which are interconnected to each other and help propagate transactions and validate transactions throughout the network. This network uses the digital currency bitcoin (BTC). These transactions are stored on a decentralized public ledger which uses blockchain technology.

The Bitcoin blockchain is made up of blocks which are linked to each preceding block, except for the first block also known as the Genesis block which isn't connected to any preceding block. Blocks are made up of several transactions which have been validated and carefully added to a block by miners, before performing the mining process. A transaction is basically a transfer of value between bitcoin wallets. Miners are responsible for performing the major work on the network which confirm or “mine” new blocks and add them to the blockchain.

There are several resources to learn about Bitcoin available online. These resources give more details about the functioning of bitcoin.

Despite the great things about bitcoin, scalability has always been a major issue. Bitcoin blocks are limited to a size of 1MB, with a block mined roughly every 10 minutes. This has put the current size of the Bitcoin blockchain at around 580 GB as of this writing, which is about 18% up over the last year. For full nodes to join the blockchain and verify networks, they need to download the entire blockchain and start performing performing validation on all old blocks and on newly added blocks. This size is about guaranteed to keep growing by the day and at some point, might become too large, so much so that it dissuades some normal users from joining the network due to it's resource constraints.

Asides the size of the blockchain, nodes also verify and store the current state of the network. This state is the current Unspent transaction output set (UTXO) which is relatively much smaller in size to the entire blockchain, however, this state as well is guaranteed to keep growing fast as more and more users carry out more transactions on the network. This set is the set of all Unspent transaction outputs in the network.

So what is Utreexo?

Utreexo introduces a hash based dynamic accumulator which allows to significantly reduce the size of the current state. It allows for nodes to fully verify inputs of a transaction without knowing the entire state of the system. It accomplishes this by letting the owner of funds maintain a proof that the funds actually do exist, and they then present this funds when they are about to spend the funds.

Utreexo introduces a new type of node called a Compact State Node. These nodes only store an accumulator representation of the state. In order for these nodes to verify transactions, they need an inclusion proof. This proof is provided by the spending transaction when they are about to spend some inputs.

How does Utreexo improve the Bitcoin network?

As seen above Utreexo allows one to represent the state of the Bitcoin network as a dynamic accumulator, these accumulators are only a few kilobytes in size, as opposed to the current state of Bitcoin, which is over 5GB large.

To understand how Utreexo works, we have to first understand what a cryptographic accumulator is, and how it works. A cryptographic accumulators, allows us to query a set without storing or revealing all members of the set. This accumulator construction approach works great for Bitcoins UXTO set because for each transaction, we would like to query whether the TXOs being spent are indeed members of the UTXO set, and if not, reject the transaction.

Regular nodes when joining the network have to download the entire blockchain history which is over 580GB and verify transactions and build their own copy of the UTXO set. They then have to verify all state changes hitting the node. All of these processes are resource intensive operations, which thereby limit the number of participants in the network which then limits scalability.

This initial synchronization process, also known as the Initial Block Download (IBD) can take a very long time, depending on internet connection and hardware resources. One of the major factors affecting the speed of this IBD operation is the type of storage disk used and the speed of I/O operations, especially, the ability to rapidly perform random access reads. This is why computers using a solid state drive, which normally has far superior random access read times can use over 30 times less time to verify the transactions, as compared to computers with a Hard disk drive.

With Utreexo, the type of disk used wouldn't make such a massive difference, as we will see just a slight performance difference between SSD computers and HDD computers

Utreexo introduces a hash-based dynamic accumulator with no trusted setup or manager requirements. As mentioned above, accumulators are compact representations of a set, to which elements can be added and proven. A Utreexo accumulator uses a forest of perfect Merkle trees which allows for efficient removals of elements from the accumulator, reducing the total number of leaves in the forest when deletions occur.

Additions are computable without any data beyond the accumulator and the element to be added, and deletions are computable with an inclusion proof of the data to be deleted.

The design of the accumulator is a forest of perfect binary hash trees. The representation of the accumulator which must be stored includes: the number of elements stored, and the root of every tree in the forest.

The logical structure of the perfect binary tree is beyond the scope of this article as this was just an introductory article. However, the full Utreexo paper can be found here.

Conclusion

Utreexo hash based accumulator aims to reduce the size of the bitcoin state to just a few kilobytes, allowing pretty much any device out there to join the bitcoin network and begin verifying transactions with no need for expensive and top shelf hardware. This will greatly increase scalability of the bitcoin network as the size of the accumulator grows very slowly (Onlogn) space complexity.
