---
layout: post
title: "From Android Developer to Bitcoin Wizard: My Summer of Bitcoin 2023 Adventure"
date: 2023-08-22
author: Prakhar Agarwal
categories: ['Stories', 'Wallets']
---

I started contributing to Summer of Bitcoin in March 2023, I already knew Android develpement but wanted to explore the blockchain world and learn about bitcoin technology. How it works? Its internals? Its ecosystem? I got across the Padawan Wallet org which was a perfect fit for me. Padawan Wallet is an android Bitcoin wallet, which is used as a learning tool by people to experiment and learn about bitcoin wallets and bitcoin ecosystem in general with testcoins. /TODO: what are testcoins. Here's a screenshot of the padawan wallet app. The padawan app is released on Playstore with early access.

My project was to develop advanced bitcoin features on the Padawan Wallet. In addition to the Padawan Wallet I contributed to the Devkit wallet as well, which is a reference application for using the bitcoindevkit library on Android.

Over the summers I had weekly 1:1 sessions with my Mentor /TODO:mention thunderbiscuit, where we discussed the progress made in the week and the next points of action. We tracked the progress on a HackMD file. //Add a pic of HackMD and discord. Our primary medium of communication was discord.

## Work Done over the summers

My project was a mix of Bitcoin and Android.

## Bitcoin

### BIP84 Descriptors

  
BIP84, also known as Bitcoin Improvement Proposal 84, is a proposal for a standard way of deriving hierarchical deterministic (HD) wallets and addresses using the BIP32 and BIP39 specifications, specifically tailored for the Segregated Witness (SegWit) address format in Bitcoin.

I updated the Devkit Wallet to use BIP84 template for descriptors

```kotlin
fun createWallet() {
        val mnemonic = Mnemonic(WordCount.WORDS12)
        val bip32ExtendedRootKey = DescriptorSecretKey(Network.TESTNET, mnemonic, null)
        val descriptor: Descriptor = Descriptor.newBip84(bip32ExtendedRootKey, KeychainKind.EXTERNAL, Network.TESTNET)
        val changeDescriptor: Descriptor = Descriptor.newBip84(bip32ExtendedRootKey, KeychainKind.INTERNAL, Network.TESTNET)
        initialize(
            descriptor = descriptor,
            changeDescriptor = changeDescriptor,
        )
        Repository.saveWallet(path, descriptor.asStringPrivate(), changeDescriptor.asStringPrivate())
        Repository.saveMnemonic(mnemonic.asString())
    }
```

### Add support for BIP21 payment links

BIP21, short for "Bitcoin Improvement Proposal 21," is a technical standard used to create and encode payment requests in URLs. It's often referred to as the "URI scheme" for Bitcoin payments. This standard allows users to easily share payment information using a simple URL format.

I updated the code to parse BIP21 URI and fetch information such as address, amount, message from it.

```kotlin
if (it != null) {
            try {
                val uri = Bip21URI.fromString(it, Network.TESTNET)
                recipientAddress.value = uri.address.value
            } catch (e: Exception) {
                recipientAddress.value = it
            }
        }
```

### Display address path when showing address

The address path is represented as a series of indices that determine the path from the master seed to a specific address. It is often depicted using a notation known as the BIP32 path format.

The BIP32 path format follows this structure:

```plaintext
codem / purpose' / coin_type' / account' / change / address_index
```

Each segment of the path represents a different level of derivation:

* `m`: Represents the master node.
    
* `purpose`: Specifies the purpose of the key derivation. Common values include 44 for Bitcoin (BIP44) and 84 for Segregated Witness (BIP84).
    
* `coin_type`: Indicates the cryptocurrency. For Bitcoin, this is usually 0.
    
* `account`: Specifies the account index. This is often used to manage different accounts within a single wallet.
    
* `change`: Denotes whether the address is for receiving (0) or change (1). Change addresses are commonly used to manage the outputs of transactions.
    
* `address_index`: The index of the address within the specified account and change type.
    

For example, an address path might look like this:

```plaintext
codem/44'/0'/0'/0/5
```

![](https://user-images.githubusercontent.com/17739006/260304174-108e49e9-c76a-47cd-9106-6408e3d43909.jpeg align="left")

### Add OP\_RETURN functionality

`OP_RETURN` is a script opcode used in Bitcoin transactions. In Bitcoin's scripting language, transactions include scripts that determine the conditions under which the funds can be spent. The `OP_RETURN` opcode is a special operation that allows data to be embedded within a transaction's output script. This data is provably unspendable, meaning it doesn't lock up any funds or require a private key to spend.

The primary use of the `OP_RETURN` opcode is to embed metadata or arbitrary data on the blockchain.

```kotlin
txBuilder = txBuilder.addData(opReturnMsg.toByteArray(charset = Charsets.UTF_8).asUByteArray().toList())
```

Here's the link to a transaction with OP\_RETURN message: [https://mempool.space/testnet/tx/249dfdeb9d77910eecd00f4e77e50848f1b53031882aed78eecb3eaf3b98ea70?showFlow=true#flow](https://mempool.space/testnet/tx/249dfdeb9d77910eecd00f4e77e50848f1b53031882aed78eecb3eaf3b98ea70?showFlow=true#flow)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692553826802/1836c276-4a3f-4c05-a71d-47ab22f23ac5.png align="center")

## Android

I polished and updated the app for its 1.0 release on Playstore.

### Migrate LiveData to Kotlin Flows

  
Kotlin Flows are a part of the Kotlin programming language's standard library, introduced to handle asynchronous programming using a reactive and more declarative approach. They are designed to provide a convenient way to work with sequences of asynchronous or potentially asynchronous values.

/insert code snippet

### Fee rate should start at 1 instead of 0

Instead of starting the fee rate from zero, we needed to start the slider from one.

### Make the “Finish” button of the same size in chapters

The Finish button was not consistent with other next and previous buttons, so I fixed it to be similar to the other buttons.

### Issues related to smaller screen sizes

There were a lot of issues related to smaller screen sizes, so I made it so that the screen will be dynamic and responsive to small screen sizes as well and support a large number of devices.

### Highlight the Bottom Navigation Tab

### Send screen to camera screen transition is awkward

### Double click open fragments multiple times

### Update to the latest version of Compose

### Disable touch opacity when Bottom sheet is expanded.

## Learnings and Takeaways
