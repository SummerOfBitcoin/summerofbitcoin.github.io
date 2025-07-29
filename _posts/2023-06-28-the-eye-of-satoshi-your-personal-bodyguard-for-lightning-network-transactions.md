---
layout: post
title: "The Eye of Satoshi: Your Personal Bodyguard for Lightning Network Transactions"
author: Aniketh Paul
date: "2023-06-28 15:09:47 +0000"
categories:
  - "Tutorials"
  - "Lightning Network"
image: https://cdn.hashnode.com/res/hashnode/image/upload/v1685559673144/749f3c9b-ef51-410b-8f21-ef12059b6914.png?auto=compress,format&format=webp
---

## **Introduction**

In this blog, we will take a deep dive into Bitcoin, the Lightning Network, and the Eye of Satoshi Watchtower. Firstly, we will explore Bitcoin's decentralized nature and cryptographic foundations, revolutionizing our perception of digital currency. Next, we will uncover the Lightning Network's potential, offering near-instant, low-cost transactions that address scalability concerns. Finally, we will unveil the Eye of Satoshi Watchtower, a critical tool bolstering the security of Lightning Network transactions. Moreover, I will provide a step-by-step guide on how to set up your own **Eye of Satoshi** and start contributing to the project. So, let's deep dive into the article.

## **Bitcoin: The Decentralized Powerhouse**

At its core, Bitcoin represents a departure from the traditional financial systems we take for granted. Created by the enigmatic Satoshi Nakamoto, Bitcoin embodies the idea of ​​decentralization by breaking the centralized control of the money market by banks and governments. Running on an end-to-end network called Blockchain, Bitcoin crosses national borders and allows people to go directly to each other, eliminating the need for intermediaries.

Bitcoin's limited supply is what sets it apart from traditional fiat currencies. Unlike central banks, which have the power to create new currencies, Bitcoin has a limited supply of 21 million coins. This uncertainty allows Bitcoin to maintain its value over time regardless of high inflation, which erodes the purchasing power of traditional money. Bitcoin's impact goes beyond its role as a digital currency. It also supports the development of an ecosystem of applications and services. From exchanges and wallets to merchant adoption and financial instruments, Bitcoin has paved the way for a new wave of innovation in the financial industry.

As a result, Bitcoin has revolutionized digital currency by providing a decentralized and secure means of exchange and store of value. Let's take a look at the Lightning Network, an incredible innovation that increases transaction speed and capacity.

## **The Quest for Scalability: Introducing the Lightning Network**

With the rise in popularity of Bitcoin, it became clear that the blockchain technology underpinning it had issues with scalability and transaction speed. The Lightning Network, which promised very instantaneous transactions with low fees, arose as a second-layer method to deal with these restrictions. The Lightning Network offers a scalable and effective framework for microtransactions by establishing off-chain payment channels that function apart from the main blockchain, promoting a wider acceptance of Bitcoin as a useful medium of exchange.

**This is how it goes:** A multi-signature transaction can be made on the Bitcoin blockchain to start a payment channel between two parties. The participants in this channel can carry out several transactions without having to record each one on the main blockchain because it acts as a secure ledger. Instead, the channel records the balances of each participant's transactions. To guarantee the security and integrity of these off-chain transactions, the Lightning Network makes use of the cryptographic assurances provided by the Bitcoin blockchain that serves as its foundation.

As we evaluate Bitcoin and its revolutionary breakthroughs in the context of the rapidly evolving digital environment, security must be taken into consideration. In this case, the Eye of Satoshi Watchtower comes into play. The Eye of Satoshi Watchtower adds an additional layer of security for Lightning Network transactions, ensuring that users can dependably take part in off-chain transactions while protecting the integrity of their assets. Let's now concentrate on fully appreciating the role and design of the Eye of Satoshi Watchtower.

## **How do watchtowers work?**

An essential part of the Lightning Network ecosystem, watchtowers give an extra degree of protection to users' lightning channels. Its main responsibility is to keep an eye on the channels' health and intervene if one channel party tries to defraud the other.

So how can a watchtower determine if a channel has been compromised? The watchtower receives the information it needs from the client, one of the channel parties, in order to properly identify and respond to any breaches. The most recent transaction details, the current channel condition, and the information required to create penalty transactions are frequently included in this information. Before transmitting the data to the watchtower, the client might encrypt it to protect privacy and secrecy. This prevents the watchtower from decrypting the encrypted data unless a breach has really taken place, even if it gets the data. The client's privacy is protected by this encryption system, which also stops the watchtower from accessing private data without authorization.

## **How to set up your own Eye of Satoshi and start contributing**

The Eye of Satoshi ([RUST-TEOS](https://github.com/talaia-labs/rust-teos?ref=blog.summerofbitcoin.org)) is a non-custodial Lightning watchtower compliant with [BOLT 13](https://github.com/sr-gi/bolt13/blob/master/13-watchtowers.md?ref=blog.summerofbitcoin.org). It has two main components:

1. **teos**: including a CLI and the server-side core functionality of the tower. Two binaries—teosd and teos-cli—will be produced when this crate is built.
2. **teos-common**: Including shared server-side and client-side functionality (helpful for creating a client).

To run the tower successfully, you will need **bitcoind** running before running the tower with the **teosd** command. Before running both of these commands you need to set up your bitcoin.conf file. Here are the steps on how to do this:-

1. Install bitcoin core from the source or [download](https://bitcoin.org/en/download?ref=blog.summerofbitcoin.org) it. After downloading, place bitcoin.conf file in the Bitcoin core user directory. Check this [link](https://en.bitcoin.it/wiki/Data_directory?ref=blog.summerofbitcoin.org) for more information regarding where to place the file as it depends on the operating system you are using.
2. After Identifying the place where the file needs to be created, add these options:-

```
#RPC
server=1
rpcuser=<your-user>
rpcpassword=<your-password>

#chain
regtest=1

```

* **server**: For RPC requests
* **rpcuser** **and** **rpcpassword**: RPC clients can authenticate with the server
* **regtest**: Not required but useful if you're planning for development.

rpcuser and rpcpassword need to be picked by you. They must be written without quotes. Eg:-

```
rpcuser=aniketh
rpcpassword=strongpassword

```

Now, if you run bitcoind it should start running the node.

1. For the tower part, first, you have to install teos from source. Follow the instructions given in this [link](https://github.com/talaia-labs/rust-teos/blob/master/INSTALL.md?ref=blog.summerofbitcoin.org).
2. After successfully installing teos in your system and running the tests, you can proceed with the last step which is to set up the teos.toml file in the teos user directory. The file needs to be placed in a folder called .teos (mind the dot) under your home folder. That is, for instance, `/home/<your-username>/.teos` for Linux. Once you've find the place, create a teos.toml file and set these options corresponding to the things we've changed on bitcoind.

```
#bitcoind
btc_network = "regtest"
btc_rpc_user = <your-user>
btc_rpc_password = <your-password>

```

Notice that here the username and password need to be written quoted, that is, for the same example as before:

```
btc_rpc_user = "aniketh"
btc_rpc_password = "strongpassword"

```

Once you've done that, you should be good to go to run the tower. Given we are running on regtest, chances are there won't be any blocks mined in out bitcoin test network the first time the tower connects to it (it there are, something is certainly off). The tower builds an internal cache of the latest 100 blocks from `bitcoind`, so on running for the first time we may get the following error:

> ERROR [teosd] Not enough blocks to start the tower (required: 100). Mine at least 100 more

Given we are running on regtest we can mine block by issuing an RPC command, without the need of waiting for the 10-minute median time that we usually see in other networks (like `mainnet` or `testnet`). Check `bitcoin-cli help` and look for how to mine blocks. If you need some help, you can go through this [article](https://developer.bitcoin.org/examples/testing.html?ref=blog.summerofbitcoin.org).

<figure>
<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1685559673144/749f3c9b-ef51-410b-8f21-ef12059b6914.png?auto=compress,format&format=webp"/>
<figcaption>That's it, you've successfully run the tower. Congratulations. 🎉</figcaption>
</figure>
