---
layout: post
title: "The Engineering Marvel of Bitcoin: Partially Signed Transactions"
date: 2023-08-12
author: Subhradeep Chakraborty
categories: ['Stories', 'Rust']
---

Bitcoin, one of the greatest inventions of this century, has been widely
accepted as an unimaginable marvel of engineering. It lets you make
transactions in a highly secure, truly decentralized, and trustless way. Among
numerous impressive features Bitcoin offers, Partially Signed Transactions
even allow users to collaboratively create transactions without the need of
being online.

# What are Partially Signed Bitcoin Transactions?

**PSBT** or **Partially Signed Bitcoin Transaction** is an interchange format
designed to simplify collaborative workflows in Bitcoin transactions. It
serves as a standardized framework for unsigned transactions, facilitating
easy cooperation among multiple parties such as hardware wallets, multi-sig
setups, and CoinJoin transactions. PSBTs enable portability and allow
participants to contribute inputs to the same transaction, simplifying the
signing process. By streamlining the creation, funding, and broadcasting of
Bitcoin transactions, PSBTs enhance flexibility and efficiency in Bitcoin
workflows.

These are the following use cases of PSBTs aim to provide-

  * **Interoperability:** PSBTs were initially developed to enhance the interoperability of Bitcoin wallets and software, fostering greater portability of transactions across different platforms. This standardized format has gained widespread industry adoption, with major wallet providers and node software embracing its benefits.
  * **Offline Signing:** One notable advantage of the PSBT format is its support for offline signing. By including essential metadata, it enables cold devices to verify addresses and amounts in a transaction, bolstering the security of signing transactions from cold storage. This streamlined process allows for the creation of transactions using a watch-only wallet, subsequent signing with a cold wallet, and final broadcasting through a Bitcoin node.
  * **Multi-signature Signing:** Moreover, PSBT simplifies multi-signature signing by rendering partially signed transactions recognizable and portable. This feature enables easy and secure signing by multiple parties or devices, enhancing the usability of multi-signature transactions. The user-friendly implementation of multi-signature capabilities contributes to improved privacy, heightened security, and enhanced loss prevention within the Bitcoin community.
  * **Multi-party transaction:** PSBT proves particularly beneficial for facilitating coordination among parties involved in signing the same transaction. Protocols like CoinJoin, CoinSwap, and PayJoin require the participation of multiple parties, all of whom must sign a shared transaction. The PSBT format provides an efficient method for constructing such transactions, facilitating their seamless passage between signers, and ultimately assembling the final transaction ready for broadcasting.

Overall, PSBTs bring about enhanced interoperability, offline signing
security, improved multi-signature functionality, and streamlined multi-party
transactions, making them a valuable assets in the realm of Bitcoin
transactions.

# How do they work?

Till now there are in total two versions of PSBT proposed by _Andrew Chow -_

  * BIP 174 : Known as PSBTV0 (PSBT version 0). This was the first-ever proposal to standardize the PSBT implementation across different wallets. Supported by all major wallets, libraries, and the bitcoin core.
  * BIP 370 : Known as PSBTV2 (PSBT version 2). The earlier version had some caveats like additional inputs and outputs couldn’t be added after the PSBT was created. The newer version addresses those issues and also offers some efficient improvements to the first version. It is not widely supported yet, but work is in progress in most cases. A Pull Request is already being actively updated to add support for PSBTV2 in the Bitcoin core.

## The overall workflow of PSBTV0

The process of constructing a fully signed Bitcoin transaction involves
several key stages:

  * **Creator** : The transaction creation begins when a proposer puts forward a specific transaction concept. They initiate a PSBT without any additional metadata, outlining the inputs and outputs. The Creator takes an unsigned transaction with all the inputs and outputs included (not signed yet!) and initializes a PSBT out of it.
  * **Updater** : Each input of the PSBT undergoes an update phase. An updater adds pertinent details about the UTXOs (Unspent Transaction Outputs) being spent, including information about the associated scripts and public keys. This step may also involve updating information related to the outputs.
  * **Signer** : Signers review the transaction and its metadata to evaluate its validity. They analyze the UTXO amounts to assess values and fees. If in agreement, they generate partial signatures for the relevant inputs based on their corresponding keys.
  * **Finalizer** : The finalization step is performed for each input. It converts the partial signatures, along with potential script information, into a final scriptSig and/or scriptWitness.
  * **Extractor** : The extractor takes the fully finalized PSBT, where all inputs are signed, and generates a valid Bitcoin transaction in network format.

Typically, these steps progress sequentially, passing the PSBT from one role
to the next until it can be transformed into an actual transaction. However,
to enable parallel operation, combiners can be employed. Combiners merge the
metadata from different PSBTs related to the same unsigned transaction,
facilitating efficient collaboration.

The roles mentioned above (Creator, Updater, Signer, Finalizer, Extractor) are
defined in BIP174 and provide a framework for understanding the underlying
process. In practical software and hardware implementations, multiple roles
are often carried out simultaneously.

## The overall workflow of PSBTV2

Since PSBTV2 introduces breaking changes to the existing PSBTV0 standards, the
workflow of PSBTV2 from creation to finalization is a little different -

  * **Creator** : The major difference between PSBTV0 and PSBTV2 is that PSBTV2 does not include any unsigned transactions. The Creator simply initializes a PSBT with 0 inputs and outputs. It also needs to set the transaction modification flags that are checked whenever modifying the PSBT.
  * **Constructor:** Only present in PSBTV2. It is responsible to add any inputs or outputs to the PSBT. Inputs are added only when the input modification flag is set to true, similarly, Outputs are added when the output modification flag is set to true. The inputs themselves contain the information regarding UTXOs (e.g. previous transaction ID, UTXO index etc.) and the outputs contain the amount being spent and the output script.  
Hence, unlike PSBTV0, all the necessary information is available in the inputs
and outputs themselves, and no need to check unsigned transactions again for
necessary details. The absence of unsigned transactions gives the freedom of
the Constructor to add as many inputs and outputs as required even after
initializing the PSBT.

  * **Updater** : In PSBTV2, an Updater updates the sequence number.
  * **Signer** : Signers sign the inputs and update the transaction modification flags to accurately reflect the state of the PSBT.
  * **Finalizer** : This step is similar to that of PSBTV0.
  * **Extractor** : Since PSBTV2 does not store unsigned transactions internally, the Extractor first needs to build the transaction using various fields of the PSBTV2. If all inputs are complete, the Extractor produces fully validated, network-serialized transactions that can be transmitted through the Bitcoin network.

Here in the article, I have shared a brief introduction to PSBTs and how they
are used in collaborative transactions. In the upcoming articles, I will go
deeper into it and give more insightful details regarding its implementation.
