---
layout: post
title: HTLCs in Lightning Network
author: Goutam Verma
date: "2022-08-19 13:32:57 +0000"
categories:
  - "Lightning Network"
  - "Tutorials"
image: https://user-images.githubusercontent.com/66783850/180373306-79799a71-aa52-469b-a086-0b9d7f518b19.jpg
---

Before we dive into the HTLC lets get a brief introduction about Lightning Network

### Lightning Network(LN)

A lightning network is a decentralized network, allowing thousands of transactions per second. It is the second layer added to the bitcoin blockchain and performs transactions between parties of the Bitcoin Blockchain.

The main aim of this network is to speed up or extend the limit of transactions in bitcoin. This network allows users to create payment channels and make transactions away from the main blockchain,  
and give benefits of blockchain's security.

![LN](https://user-images.githubusercontent.com/66783850/180373306-79799a71-aa52-469b-a086-0b9d7f518b19.jpg)

### HTLC

HTLC stands for Hash Time Lock Contracts, It doesn't require trusted third parties. In this type of contract payments are generated with a "time lock". The structure of HTLC contains two parts

* Hash verification
* Time expires verification.

Let's start with the hash, To make HTLC payment the secret message "M" should be created at first, and then its hash should be calculated using a hash function.

`H = Hashing(M)`

This hash "H" will be included in the locking script. Therefore, only the person who knows the secret("M") that hashed to get H will be able to use the payment. The secret M is known as `preimage`.

The second part of the HTLC contract is the verification of the lock time for the payment. If the secret was not revealed in time and the payment was not used, the sender will be able retrieve all the funds.

To open a hash lock and claim a payment, the receiver needs to reveal the `preimage` to a hash digest encoded in the contract.  
To open a time lock and receive a refund, the spender needs to wait until after a certain time encoded in the contract.

### LN Example

Now we have understanding of basic components of HTLC, let's understand the complete process through scenario.

* Three persons alice, bob and caroline.
* Alice has open channel to bob and bob has open payment channel to caroline.

![person](https://user-images.githubusercontent.com/66783850/180373333-947d46e4-6d06-45ac-89df-2bdea20a1428.png)

Alice wants to buy something from Caroline for 100 mBTC, Alice open a payment channel to Bob, and Bob opens a payment channel to Caroline, but Alice doesn't have direct open channel with Caroline.

To make a successful payment between Alice and Caroline, Alice uses a HTLC in the following manner.

* Caroline generates a random hash "H" with help the of any message "M". Caroline gives that hash to Alice.

![caroin](https://user-images.githubusercontent.com/66783850/180373353-5d19a433-794d-430f-b540-052d76c2f0d2.png)

* Whereas Alice added hash given by caroline and make payment of 100 mBTC to bob (In order for bob to claim the payment, he needs the message which was used to produce that hash).

![alicebob](https://user-images.githubusercontent.com/66783850/180373374-0b19679f-fdbe-4102-b23b-4eede1c1dccc.png)

* Bob uses his payment channel to pay 100 mBTC to caroline, and he also adds the copy of conditions that alice put on when she gave payment to bob.(as similar to above step)
* Caroline has the original message that was used to produce the hash(also called a pre-image). At last Caroline uses message to finalize payment and fully receive the payment from bob, By doing this Caroline need to make the pre-image available to Bob.

![preimage](https://user-images.githubusercontent.com/66783850/180373407-049c0e80-b1ee-444e-91ab-a61d52aab8c3.png)

* Bob uses the pre-image to finalize his payment from Alice.

On this way, Alice paid 100 mBTC without establishing a new channel that would link them directly. No one of the chain participants was obliged to trust others.  
inside the case of every body from the chain failing to perform there a part of work, nobody would have lost the cash that would honestly stay frozen until the expiration of the lock time.

### Test Failures

In our testcase, everything went perfectly but in real world everything would obey, and if something can go wrong, it would go wrong, so let's discuss the `protection` mechanisms of the Lightning Network (LN).

Longer the chain used to deliver the funds, the higher the probability of not succeeding

* Some participants may close the channel
* Some may simply lose the Internet connection.

Let us imagine that the funds have successfully reached the end of the chain, but on the way back (when the secret should follow the backtrace) a problem has emerged when one of the participant refused or was unable to cooperate.  
In our example, it would be Bob.

![channelclose](https://user-images.githubusercontent.com/66783850/180373432-cab80473-4ef1-40fb-902f-dd9566d0e3a0.png)

Failure in funds delivery due to one of the channels closing,

* Caroline: when the chain has reached her. She immediately retrieves her funds using the secret and revealing it to Rebeca.
* Rebeca: wishes to get her money back from Bob, but he does not respond so to avoid risk she closes the channel and send the last commitment transaction (the HTLC contract previously sent by Bob) into the blockchain and, with the help of the secret, retrieves the funds.
* Bob: still has three days to surface and took his money from Alice (as the transaction has made it to the blockchain, he can easily find it and see the R).
* Otherwise, at the expiration of the lock time, Alice will be able to retrieve all his money.

Note : It can seem that if a participant leaves the system for some reason, he or she is the only one who risks losing funds, while all other participants of the chain are safe.
