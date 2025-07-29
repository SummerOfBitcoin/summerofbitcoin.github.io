---
layout: post
title: "My Summer of Bitcoin Journey: From Learning the Ropes to Contributing to Blockcore"
date: 2024-06-28
author: Dhruv Bhanushali
categories: [Blockcore, Stories, "2024"]
image: ""
---

Okay here we go.., this is my first time publishing something so please forgive me for the mistakes🤧

## Introduction to SOB

Prior to Summer of Bitcoin, I knew only a few things about the crypto space, mostly from the hype during the COVID-19 times. Like everyone else, I didn't know exactly what it was.

As a developer, I was looking to explore new things and domains. Through friends and seniors, I found out about Summer of Bitcoin. And with that i thought, "Oh well, this is a great chance to learn what all the hype around Bitcoin really is..."

## Applications

The application process was really simple, just fill out the details, github, resume and basic skill set, and a reasoning for why you are interested, simple enough.

After a few days i got the selection email, and was told to introduce myself in the discord server.

# Learning Phase

We were given many learning resources on the website, and after exploring all the ways, i started learning from the [mastering bitcoin book](https://github.com/bitcoinbook/bitcoinbook), its a great resource for someone coming with zero prior information.

I learnt about the reasoning behind why a decentralized currency is such a big deal, what was the purpose for which Satoshi Nakamoto created Bitcoin,

From a technical standpoint it was amazing to get to know how thousands of computers communicate in real-time and how the consensus happens, the robust networking and cryptographic measures taken to make bitcoin safe.

Its fascinating to see a truly democratic currency work, how each member of the community has a vote in the direction of where the project goes...

Alongside the book, there were discussion sessions every week where we can discuss what we learnt from the book, help each other and discuss in general.

So after this i can say i have learnt a lot about Bitcoin's core functionality

## Assignment

After the learning, discussion and understanding the theoretical aspects of bitcoin, it was time to know what it is to code; {spoiler: its a bit hard lol}

ps this was also a great chance to use the rust, which i had learnt recently

### Goal

Simply speaking, our goal in the assignment was to create a bitcoin block from the list of transactions given as JSON in a human readable form, We had to convert the transactions to the format used in Bitcoin, verify the transactions and then create a block out of them.

```json
{
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "3b7dc918e5671037effad7848727da3d3bf302b05f5ded9bec89449460473bbb",
      "vout": 16,
      "prevout": {
        "scriptpubkey": "0014f8d9f2203c6f0773983392a487d45c0c818f9573",
        "scriptpubkey_asm": "OP_0 OP_PUSHBYTES_20 f8d9f2203c6f0773983392a487d45c0c818f9573",
        "scriptpubkey_type": "v0_p2wpkh",
        "scriptpubkey_address": "bc1qlrvlygpudurh8xpnj2jg04zupjqcl9tnk5np40",
        "value": 37079526
      },
      "scriptsig": "",
      "scriptsig_asm": "",
      "witness": [
        "30440220780ad409b4d13eb1882aaf2e7a53a206734aa302279d6859e254a7f0a7633556022011fd0cbdf5d4374513ef60f850b7059c6a093ab9e46beb002505b7cba0623cf301",
        "022bf8c45da789f695d59f93983c813ec205203056e19ec5d3fbefa809af67e2ec"
      ],
      "is_coinbase": false,
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "scriptpubkey": "76a9146085312a9c500ff9cc35b571b0a1e5efb7fb9f1688ac",
      "scriptpubkey_asm": "OP_DUP OP_HASH160 OP_PUSHBYTES_20 6085312a9c500ff9cc35b571b0a1e5efb7fb9f16 OP_EQUALVERIFY OP_CHECKSIG",
      "scriptpubkey_type": "p2pkh",
      "scriptpubkey_address": "19oMRmCWMYuhnP5W61ABrjjxHc6RphZh11",
      "value": 100000
    },
    {
      "scriptpubkey": "0014ad4cc1cc859c57477bf90d0f944360d90a3998bf",
      "scriptpubkey_asm": "OP_0 OP_PUSHBYTES_20 ad4cc1cc859c57477bf90d0f944360d90a3998bf",
      "scriptpubkey_type": "v0_p2wpkh",
      "scriptpubkey_address": "bc1q44xvrny9n3t5w7lep58egsmqmy9rnx9lt6u0tc",
      "value": 36977942
    }
  ]
}
```

I am not going into technical details here but giving a short summary of the steps below :)

### Journey from a pool of Txns to a Block :

First steps were knowing how the transaction is structured to be stored in Blockchain. This is really important as the structure should be such as there is a minimal amount of redundant storage used, this will make the storage as well as sharing of transaction efficient.

There are lots of specifics involved in the serialization of different parts of the transaction, some are little-endian, some are big-endian, after the introduction of segwit many parts have become reusable etc

```plaintext

01000000000101bb3b4760944489ec9bed5d5fb002f33b3dda278784d7faef371067e518c97d3b1000000000ffffffff02a0860100000000001976a9146085312a9c500ff9cc35b571b0a1e5efb7fb9f1688ac163d340200000000160014ad4cc1cc859c57477bf90d0f944360d90a3998bf024730440220780ad409b4d13eb1882aaf2e7a53a206734aa302279d6859e254a7f0a7633556022011fd0cbdf5d4374513ef60f850b7059c6a093ab9e46beb002505b7cba0623cf30121022bf8c45da789f695d59f93983c813ec205203056e19ec5d3fbefa809af67e2ec00000000
```

This is how the above transaction looks like after getting serialized.

[Learn me a bitcoin](https://learnmeabitcoin.com/) -> one of the best places to learn about the structure of bitcoin blocks.

After the transactions are serialized, we had to verify if the transactions are valid or not, by matching the signatures with the serialized transactions ( not exactly transactions but pre-image ), using the ECDSA signatures and stuff, got to learn a lot about digital signatures, asymmetric keys etc, it's really awesome how a currency that is completely open-source and with nothing but the private keys in secret remains so much secure.

We can determine the weight of the transaction from the number of bytes it takes, a Bitcoin block is 1mb (witness data is separate tho), now we select the required number of transaction from the mempool and proceed to the next steps.

Now that we have the transactions, we can create the merkle-root from the txids, and from it the block-header and boom, our block is done !!!


## Proposal Round

After completing the assignment, we were instructed to choose a project that interests us from a list of various organizations, then research about the project and prepare a proposal on the approach you will be taking to solve the problem.

I chose to work with [Blockcore](https://www.blockcore.net/) on the project, Migrating Indexer from MongoDB to Postgre, where i will be working with C# .NET core and PostgreSQL.

I wrote the timeline of how i will be working upon the project, started familiarizing myself with the codebase and the pattern, communicated with the mentors on parts which were not clear to me etc.

# Project Phase

Blockcore aims to provide building blocks to create decentralized software, I will be working on **Blockcore** **Indexer** (blockchain API) that builds a database of history, including tokens, NFTs, transactions and more. where I will be working on adding support for relational databases (PostgreSQL at the moment) and aim to improve ingestion and query performance.

### Contributing To Blockcore Indexer

You can start contributing to the Indexer by first looking at the schema on [Github](https://github.com/block-core/blockcore-indexer)(and also join the Discord), take a look at the DI services for the web-host in startup.cs, more of the working of data-store is completely decoupled now, you can understand how it works by exploring the Storage namespace

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724857800739/7c0b1be5-6c6c-46ba-b92a-c099736bba8b.png align="center")

\*\* some changes may not be reflected until my code reaches prod :)

For interacting with the BTC nodes the Indexer mostly uses the Blockcore's own extensions alongside NBitcoin, Its quite straightforward to start working with the code

## Things i worked upon during the Project

First of all, as I was tasked with implementing a completely new feature, my first step was to create a prototype to benchmark performance using EFCore and Dapper. EFCore was chosen for its ease of use and SQLBulkCopy extensions, which improved ingestion performance. While working with the prototype, I familiarized myself with Blockcore extensions and gained a bit of insights into how the indexer functions. Although the prototype was a mock-up and didn’t include actual queries, it was good enough for testing data insertion performance under various conditions. This gave me a clear idea of the work needed on the migration.

Coming to the main project, most of my work focused on the Storage Namespace. I created new classes for operations on PostgreSQL, according to the interfaces. I integrated EFCore, set up the DBContext, and created new types and schemas for the relational model.

Various changes were needed to ensure smooth integration with the new database. We did some decoupling to remove dependencies on MongoDB and allow for easy switching between both databases. Most of the refactoring was led by my mentor, with my primary role being to identify problematic couplings.

// some of the files changed/introduced

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724877036465/43669146-31fc-4453-a1cc-f75ab28fc7a7.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724877079178/e4832ff3-a8a4-4b54-9a63-ea87df372157.png align="center")

.... This is the `IStorage` interface, which provides functionalities for controllers, relying on `IStorageOperations` for tasks like adding, deleting, and handling storage batches. And both of these are used by the runner tasks to create new batches by fetching data from the node to be put in the Indexer. The process was generally straightforward, though, as mentioned, some decoupling was necessary before everything could work seamlessly together.([link to refractoring changes](https://github.com/block-core/blockcore-indexer/commit/e27a2c918e200fb6d040e0e7c679184f986912af)).

After checking, fixing and iterating the work was done!!

### What I Learnt

So after the completion of the project i had implemented many of the things that i read in the bitcoin book, and got to understand how to approach to fixing things one at a time on a larger code base, to write maintainable code & learned a bit of good refractoring practices and design patterns.

## Conclusion

Summer of Bitcoin was a great experience where i got to meet new people, learn about a emerging technology and create code which will be used by lots of people!!!
