---
layout: post
title: "My Summer of Bitcoin Journey: From Zero to Bitcoin Contributor with Blockcore"
date: 2024-07-15
author: "Dhruv Bhanushali"
categories: [Stories, Blockcore]
---

Okay here we go.., this is my first time publishing something so please forgive me for the mistakes🤧

### Introduction to SOB

Prior to Summer of Bitcoin, I knew only a few things about the crypto space, mostly from the hype during the COVID-19 times. Like everyone else, I didn't know exactly what it was.

As a developer, I was looking to explore new things and domains. Through friends and seniors, I found out about Summer of Bitcoin. And with that i thought, "Oh well, this is a great chance to learn what all the hype around Bitcoin really is..."

### Applications

The application process was really simple, just fill out the details, github, resume and basic skill set, and a reasoning for why you are interested, simple enough.

After a few days i got the selection email, and was told to introduce myself in the discord server.

### Learning Phase

We were given many learning resources on the website, and after exploring all the ways, i started learning from the mastering bitcoin book, its a great resource for someone coming with zero prior information.

I learnt about the reasoning behind why a decentralized currency is such a big deal, what was the purpose for which Satoshi Nakamoto created Bitcoin, From a technical standpoint it was amazing to get to know how thousands of computers communicate in real-time and how the consensus happens, the robust networking and cryptographic measures taken to make bitcoin safe.

Its fascinating to see a truly democratic currency work, how each member of the community has a vote in the direction of where the project goes...

Alongside the book, there were discussion sessions every week where we can discuss what we learnt from the book, help each other and discuss in general.

So after this i can say i have learnt a lot about Bitcoin's core functionality

### Assignment

After the learning, discussion and understanding the theoretical aspects of bitcoin, it was time to know what it is to code; {spoiler: its a bit hard lol}

ps this was also a great chance to use the rust, which i had learnt recently

#### Goal

Simply speaking, our goal in the assignment was to create a bitcoin block from the list of transactions given as JSON in a human readable form, We had to convert the transactions to the format used in Bitcoin, verify the transactions and then create a block out of them.

I am not going into technical details here but giving a short summary of the steps below :)

#### Journey from a pool of Txns to a Block :

First steps were knowing how the transaction is structured to be stored in Blockchain. This is really important as the structure should be such as there is a minimal amount of redundant storage used, this will make the storage as well as sharing of transaction efficient.

There are lots of specifics involved in the serialization of different parts of the transaction, some are little-endian, some are big-endian, after the introduction of segwit many parts have become reusable etc

This is how the above transaction looks like after getting serialized.

Learn me a bitcoin -> one of the best places to learn about the structure of bitcoin blocks.

After the transactions are serialized, we had to verify if the transactions are valid or not, by matching the signatures with the serialized transactions ( not exactly transactions but pre-image ), using the ECDSA signatures and stuff, got to learn a lot about digital signatures, asymmetric keys etc, it's really awesome how a currency that is completely open-source and with nothing but the private keys in secret remains so much secure.

we can determine the weight of the transaction from the number of bytes it takes, a Bitcoin block is 1mb (witness data is separate tho), now we select the required number of transaction from the mempool and proceed to the next steps.

Now that we have the transactions, we can create the merkle-root from the txids, and from it the block-header and boom, our block is done !!!

So this was the overview of the process for completing the assignment, you can take a look at the complete code on my github.

-> Assignment Repository

### Proposal Round

After completing the assignment, we were instructed to choose a project that interests us from a list of various organizations, then research about the project and prepare a proposal on the approach you will be taking to solve the problem.

I chose to work with Blockcore on the project, Migrating Indexer from MongoDB to Postgre, where i will be working with C# .NET core and PostgreSQL.

I wrote the timeline of how i will be working upon the project, started familiarizing myself with the codebase and the pattern, communicated with the mentors on parts which were not clear to me etc.

### Project Phase

Blockcore aims to provide building blocks to create decentralized software, I will be working on Blockcore Indexer (blockchain API) that builds a database of history, including tokens, NFTs, transactions and more. where I will be working on adding support for relational databases (PostgreSQL at the moment) and aim to improve ingestion and query performance.

#### Contributing To Blockcore Indexer

You can start contributing to the Indexer by first looking at the schema on Github(and also join the Discord), take a look at the DI services for the web-host in startup.cs, more of the working of data-store is completely decoupled now, you can understand how it works by exploring the Storage namespace

** some changes may not be in effect until my code reaches prod :)

For interacting with the BTC nodes the Indexer mostly uses the Blockcore's own extensions alongside NBitcoin, Its quite straightforward to start working with the code

#### Things i worked upon during the Project

First of all, as I was tasked with implementing a completely new feature, my first step was to create a prototype to benchmark performance using EFCore and Dapper. EFCore was chosen for its ease of use and SQLBulkCopy extensions, which improved ingestion performance. While working with the prototype, I familiarized myself with Blockcore extensions and gained a bit of insights into how the indexer functions. Although the prototype was a mock-up and didn't include actual queries, it was good enough for testing data insertion performance under various conditions. This gave me a clear idea of the work needed on the migration.

Coming to the main project, most of my work focused on the Storage Namespace. I created new classes for operations on PostgreSQL, according to the interfaces. I integrated EFCore, set up the DBContext, and created new types and schemas for the relational model.

Various changes were needed to ensure smooth integration with the new database. We did some decoupling to remove dependencies on MongoDB and allow for easy switching between both databases. Most of the refactoring was led by my mentor, with my primary role being to identify problematic couplings.

// some of the files changed/introduced

.... This is the IStorage interface, which provides functionalities for controllers, relying on IStorageOperations for tasks like adding, deleting, and handling storage batches. And both of these are used by the runner tasks to create new batches by fetching data from the node to be put in the Indexer. The process was generally straightforward, though, as mentioned, some decoupling was necessary before everything could work seamlessly together.(link to refractoring changes).

After checking, fixing and iterating the work was done!!

### What I Learnt

So after the completion of the project i had implemented many of the things that i read in the bitcoin book, and got to understand how to approach to fixing things one at a time on a larger code base, to write maintainable code & learned a bit of good refractoring practices and design patterns.

### Conclusion

Summer of Bitcoin was a great experience where i got to meet new people, learn about a emerging technology and create code which will be used by lots of people!!!
