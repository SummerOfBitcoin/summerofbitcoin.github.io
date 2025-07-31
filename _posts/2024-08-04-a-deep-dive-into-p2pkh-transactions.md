---
layout: post
title: A Deep Dive into P2PKH Transactions
date: 2024-08-04
author: Vanshul Bhatia
categories: [Tutorials, Bitcoin]
image: https://miro.medium.com/v2/resize:fit:1400/1*45weRC1vzYsEvG1Y3w1qrA.png
---

The main motivation to get started with Bitcoin Development and understanding transaction was through the Summer of Bitcoin Cohort. We need to build a transaction validator from scratch, thus I was going deep into UTXOs and P2PKH transactions.

The goal of this blog is to give a thorough introduction to the most common type of Bitcoin transactions, *Pay-to-Public-Key-Hash* (P2PKH). To achieve this, the following is presented:

* A succinct overview of the concept of an unspent transaction output (UTXO) and how a transaction is formed
    
* A breakdown of each piece of a P2PKH transaction
    

A P2PKH transaction is the type of transactions that most people make when they move a specified amount of Bitcoin from one address to another usually via a wallet interface. An example transaction is used to clearly demonstrate part of what is going on behind the scenes of an average wallet.

# Unspent Transaction Outputs (UTXOs)

A very important concept to understand before diving into the transaction breakdown is that of an *unspent transaction output* or UTXO. In a Bitcoin transaction, UTXOs are what is being consumed, or spent. A UTXO can only be spent once. After it is spent it is then referred to as a *spent transaction output*.

Spending a UTXO is similar to taking a $20 bill to buy something for $10 but instead of the cashier keeping the $20 bill, she lights it on fire and creates two new $10 bills out of thin air, keeping one and returning the other to you as change.

In this analogy the incinerated $20 bill started as a UTXO but once it was consumed became a spent transaction output which spawned two new $10 bills representing two new UTXOs ready to be spent in future transactions.

All the Bitcoin available for consumption on the network are referred to as the *UTXO set*.

# Transaction Building Process

Each transaction consists of the *version number*, *input*, *output*, and *locktime*.

The input contains an *outpoint(s)*, a *sequence number*, and an *unlocking script*, also called the *scriptSig*.

The output contains the *value* of the amount being spent and the *locking script*, also referred to as the *scriptPubKey*.

The locktime dictates at what time the transaction becomes valid.

Every transaction has a minimum of one input and one output. The inputs contains logic that tells the network which UTXOs to consume (via the outpoints) and proves that it is allowed to consume them (via the unlocking scripts). The output contains logic that tells the network the conditions in which a future transaction is allowed to consume the newly created UTXOs (via the locking script).

The figure below aims to map out the relationship between confirmed transaction with spent outputs, confirmed transactions with unspent outputs, and new, unsubmitted transactions.

![Transaction Relationship](https://miro.medium.com/v2/resize:fit:1400/1*45weRC1vzYsEvG1Y3w1qrA.png align="left")

## Steps to Build a Transaction

The steps to build a P2PKH transaction are as follows:

1. Identify a previous transaction that contains UTXOs that you have control over (just some Bitcoin that you already own).
    
2. Build the input outpoints of the new transaction to identify the previous transaction’s UTXO to spend.
    
3. Build the outputs of the new transaction so the locking script will contain the conditions in which the newly created UTXOs can be spent/unlocked by the next transaction.
    
4. Create the unlocking script so it meets the conditions set by the previous transactions output’s locking script. This contains the recipient’s signature and is created at the very last step, but is very much a part of the input and is physically placed in the middle of the transaction.
    

# Introduction to the Serialized Transaction

A complete signed P2PKH Bitcoin transaction is discussed in detail below.

This transaction was randomly chosen from [Blockchain.com](http://Blockchain.com) and can be viewed [here](https://www.blockchain.com/en/btc/tx/e65ad475a01384b086ce0d04199835fdd580739422ece1e0f1c4e362d43735d9) [(ra](https://www.blockchain.com/en/btc/tx/e65ad475a01384b086ce0d04199835fdd580739422ece1e0f1c4e362d43735d9)[w values](https://blockchain.info/rawtx/e65ad475a01384b086ce0d04199835fdd580739422ece1e0f1c4e362d43735d9)[).](https://blockchain.info/rawtx/e65ad475a01384b086ce0d04199835fdd580739422ece1e0f1c4e362d43735d9)
```plaintext
01000000019c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f886
c48010000006a47304402203da9d487be5302a6d69e02a861acff1da472885e43d7528e
d9b1b537a8e2cac9022002d1bca03a1e9715a99971bafe3b1852b7a4f0168281cbd27a2
20380a01b3307012102c9950c622494c2e9ff5a003e33b690fe4832477d32c2d256c67e
ab8bf613b34effffffff02b6f50500000000001976a914bdf63990d6dc33d705b756e13
dd135466c06b3b588ac845e0201000000001976a9145fb0e9755a3424efd2ba0587d20b
1e98ee29814a88ac00000000
```
From a UTXO with a value of 17,373,066 address 19iy8HKpG5EbsqB2GUNVPUDbQxiTrPXpsx sends 390,582 satoshi to 1JKRgG4F7k1b7PbAhQ7heEuV5aTJDpK9TS and receives 16,932,484 satoshi back as change. The remaining delta between the inputs and outputs go to the miner as a transaction processing fee.

# Version Number

```markdown
`01000000`019c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f886c48010000006a47304402203da9d487be5302a6d69e02a861acff1da472885e43d7528ed9b1b537a8e2cac9022002d1bca03a1e9715a99971bafe3b1852b7a4f0168281cbd27a220380a01b3307012102c9950c622494c2e9ff5a003e33b690fe4832477d32c2d256c67eab8bf613b34effffffff02b6f50500000000001976a914bdf63990d6dc33d705b756e13dd135466c06b3b588ac845e0201000000001976a9145fb0e9755a3424efd2ba0587d20b1e98ee29814a88ac00000000
```
The version number is four bytes long and is expressed as a hexadecimal value in little endian format.

There are two version types. Version 01 indicates that there is no relative time lock. Version 02 indicates that there may be a relative time lock. Version 02 was introduced in [BIP0068](https://github.com/bitcoin/bips/blob/master/bip-0068.mediawiki) which added OP_CHECKSEQUENCEVERIFY along with [BIP0112](https://github.com/bitcoin/bips/blob/master/bip-0112.mediawiki#Bidirectional_Payment_Channels) which upgraded it. Version 02 is used in conjunction with the sequence number which is described below.

The example transaction is version 01.

# Transaction Input

```markdown
01000000`019c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f8`
`86c48010000006a47304402203da9d487be5302a6d69e02a861acff1da472885e43d75`
`28ed9b1b537a8e2cac9022002d1bca03a1e9715a99971bafe3b1852b7a4f0168281cbd`
`27a220380a01b3307012102c9950c622494c2e9ff5a003e33b690fe4832477d32c2d25`
`6c67eab8bf613b34effffffff`02b6f50500000000001976a914bdf63990d6dc33d705
b756e13dd135466c06b3b588ac845e0201000000001976a9145fb0e9755a3424efd2ba0
587d20b1e98ee29814a88ac00000000
```

Each transaction input points to a previous transactions output. If the unlocking script in the current transaction input meets the conditions set by the output script of the previous transaction, then the UTXO that the previous transaction “holds” can be spent.

## Number of Outpoints

```markdown
01000000`01`9c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f
886c48010000006a47304402203da9d487be5302a6d69e02a861acff1da472885e43d7
528ed9b1b537a8e2cac9022002d1bca03a1e9715a99971bafe3b1852b7a4f0168281cb
d27a220380a01b3307012102c9950c622494c2e9ff5a003e33b690fe4832477d32c2d2
56c67eab8bf613b34effffffff02b6f50500000000001976a914bdf63990d6dc33d705
b756e13dd135466c06b3b588ac845e0201000000001976a9145fb0e9755a3424efd2ba0587d20b1e98ee29814a88ac00000000
```

The next one to nine bytes in the transaction input defines the number of included outpoints and is of type VarInt. There is always one outpoint for each UTXO being consumed.

The example transaction as 1 UTXO as an input.

## Transaction Outpoint

```markdown
0100000001`9c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f88`
`6c4801000000`6a47304402203da9d487be5302a6d69e02a861acff1da472885e43d752
8ed9b1b537a8e2cac9022002d1bca03a1e9715a99971bafe3b1852b7a4f0168281cbd27a
220380a01b3307012102c9950c622494c2e9ff5a003e33b690fe4832477d32c2d256c67e
ab8bf613b34effffffff02b6f50500000000001976a914bdf63990d6dc33d705b756e13d
d135466c06b3b588ac845e0201000000001976a9145fb0e9755a3424efd2ba0587d20b1e
98ee29814a88ac00000000
```

Each outpoint is a reference to a previous transaction hash and a corresponding index that points to the exact UTXO that is being spent.

The first 32 bytes of the outpoint is the reference to the previous transaction hash containing the UTXO being consumed in little endian format. For example, the previous transaction hash is:

```plaintext
486c887f2378feb1ea3cdc054cb7b6722e632ab1edac962a00723ea0240f2e9c
```

and when it is included in the transaction outpoint, it expressed in little endian format(reverse byte order):

```markdown
01`9c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f886c48`01000000
```

The last four bytes of the outpoint define the index of which UTXO of the previous transaction is being consumed. It is also expressed has a hexadecimal value in little endian format.

```markdown
019c2e0f24a03e72002a96acedb12a632e72b6b74c05dc3ceab1fe78237f886c48
`01000000`
```

The transaction containing the UTXO being spent can be viewed [here](http://486c887f2378feb1ea3cdc054cb7b6722e632ab1edac962a00723ea0240f2e9c) ([raw values](https://blockchain.info/rawtx/486c887f2378feb1ea3cdc054cb7b6722e632ab1edac962a00723ea0240f2e9c)). The first entry in the previous transaction’s “out” key shows that a UTXO containing a decimal value 17,373,066 satoshi is being consumed in the current transaction. Note that the current transaction does not explicitly include the 17,373,066 satoshi UTXO, only the reference to it via the outpoint contents.

## Unlocking Script

The unlocking script, also called scriptSig, contains the stack script (signature) and redeem script (public key) of the user building the transaction and receiving the funds.

.. To Be Continued
