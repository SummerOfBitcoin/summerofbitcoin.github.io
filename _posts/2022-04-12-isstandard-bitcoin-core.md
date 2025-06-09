---
layout: post
title: Digging into IsStandard in Bitcoin Core
author: Summer of Bitcoin
date: "2022-04-12 12:19:00 +0000"
tags:
  - "Tutorials"
---

### By Arnab Sen (@ArnabSen1729) Summer of Bitcoin '21

In this post, we will focus on the Bitcoin Core implementation of the different types of scripts / transactions methods. Here is the link to the repo:

[![](https://opengraph.githubassets.com/95158c4c419edcddca5657f6fac350184b24c5cf2b2a1e0085e2c5990d8a24ca/bitcoin/bitcoin)](https://github.com/bitcoin/bitcoin?ref=blog.summerofbitcoin.org)

## Standard Types

There is an enum class called `TxoutType` that lists all the possible types of TxOut i.e Output Transaction.  
![tx-out](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/txoutype.png)

[src/script/standard.h#L59-L71](https://github.com/bitcoin/bitcoin/blob/71797beec54d36a055d5e172ecbf2141fa984428/src/script/standard.h?ref=blog.summerofbitcoin.org#L59-L71)

An enumeration is a user-defined type that consists of a set of named integral constants that are known as enumerators. In this case, any instance of the `TxoutType` can be either of those mentioned values. This in itself gives us an idea about what are the different kinds of Transaction Types. The transaction type `NONSTANDARD` is considered invalid.

> P.S In this blog I will cover the 5 valid types, will ignore the WITNESS ones they are related to SegWit. Will cover it in some other article.

Before diving into these, let's look at the function that determines if a Script is standard.

![isStandard](../assets/images/blog_content/isstandard.png)

[src/policy/policy.cpp#L53-L74](https://github.com/bitcoin/bitcoin/blob/71797beec54d36a055d5e172ecbf2141fa984428/src/policy/policy.cpp?ref=blog.summerofbitcoin.org#L53-L74)

This `isStandard` function is a boolean function that returns true if a locking script is standard or false. In the first two lines:

```
std::vector<std::vector<unsigned char> > vSolutions;
whichType = Solver(scriptPubKey, vSolutions);

```

**vSolutions** is a vector that stores the important data of the script excluding the opcodes. We will see later on which data is pushed into this vector.

**Solver** is a helper function that does the job of determining the type of script. It accepts `vSolutions` as a reference and updates the vector with the important data of the script, and then returns a `TxoutType` value.

Let's look into the `Solver` function. It's a huge function, but we will look at it section by section. Here is a glance at the entire function:

![solver](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/solver.png)

[src/script/standard.cpp#L144-L211](https://github.com/bitcoin/bitcoin/blob/71797beec54d36a055d5e172ecbf2141fa984428/src/script/standard.cpp?ref=blog.summerofbitcoin.org#L144-L211)

Let's get started with our first type of Transaction.

### `Pay to Public Key Hash (P2PKH)`

The vast majority of transactions processed on the bitcoin network spend outputs locked with a Pay-to-Public-Key-Hash or "P2PKH" script.

| Script | Contains |
| --- | --- |
| **Locking Script** | A public key hash, more commonly known as a bitcoin address |
| **Unlocking Script** | A public key and a digital signature created by the corresponding private key |

So let's say if Alice wants to send some coins to Bob, the output transaction will contain a locking script of the format:

```
OP_DUP OP_HASH160 20 <Bob's Public Key Hash> OP_EQUALVERIFY OP_CHECKSIG

```

Now, when Bob will have to spend the coins he received from Alice he should include an Unlocking script like this:

```
<Bob's Signature> <Bob's Public Key>

```

Here is how the `Solver` determines if a transaction is of P2PKH type:

![MatchP2PKH.png](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/MatchP2PKH.png)

It calls a helper function of `MatchPaytoPubkeyHash`.

![MatchP2PKH](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/MatchP2PKH-1.png)

[src/script/standard.cpp#L79-L86](https://github.com/bitcoin/bitcoin/blob/71797beec54d36a055d5e172ecbf2141fa984428/src/script/standard.cpp?ref=blog.summerofbitcoin.org#L79-L86)

Let's look at the locking script once again:

```
OP_DUP OP_HASH160 20 <Public Key Hash> OP_EQUALVERIFY OP_CHECKSIG

```

Every Opcode occupies 1 byte. And the number 20 represents the size of the Hash. It also occupies 1 byte. The Hash itself occupies 20 bytes. So the total size is 4 byte (1 for each opcode) + 1 byte (for the size) + 20 byte hash = 25 bytes.

In the `MatchPayToPubkeyHash` it first checks if the size matches, and then checks if the first 3 values should be `OP_DUP`, `OP_HASH160` and `20`. And finally the last and the second last values should be `OP_CHECKSIG` and `OP_EQUALVERIFY`.

If these conditions are satisfied then it returns true else false.

![Tx_Script_P2PubKeyHash_1](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/Tx_Script_P2PubKeyHash_1.png)  
![Tx_Script_P2PubKeyHash_2](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/Tx_Script_P2PubKeyHash_2.png)

*Fig: How P2PKH script is executed by the bitcoin engine*

### `Pay to Public Key (P2PK)`

Pay-to-public-key is a simpler form of a bitcoin payment than pay-to-public-key-hash. With this script form, the public key itself is stored in the locking script, rather than a public-key-hash as with P2PKH earlier, which is much shorter. Pay-to-public-key-hash was invented by Satoshi to make bitcoin addresses shorter, for ease of use. Pay-to-public-key is now most often seen in coinbase transactions, generated by older mining software that has not been updated to use P2PKH.

| Script | Contains |
| --- | --- |
| **Locking Script** | A public key |
| **Unlocking Script** | A digital signature created by the corresponding private key |

If Alice sends Bob some BTC then Locking Script will be:

```
<Key Size> <Bob's Public Key> OP_CHECKSIG

```

Corresponding Unlocking Script for Bob will be:

```
<Bob's Signature>

```

Here is the code snippet that does the check for `P2PK`:  
![p2pk](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/p2pk.png)

If we look into the `MatchPayToPubKey` we will see a very similar check like that of P2PKH.  
There are two versions of PubKey one is the normal one and the other is the compressed version:

![MatchPayToPubKey](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/MatchPayToPubKey.png)

[src/script/standard.cpp#L66-L77](https://github.com/bitcoin/bitcoin/blob/71797beec54d36a055d5e172ecbf2141fa984428/src/script/standard.cpp?ref=blog.summerofbitcoin.org#L66-L77)

In both cases the check is very similar it makes sure that at the back we have `OP_CHECKSIG` and the size matches to `CPubKey::SIZE+2`. Why `+2`? Because 1 byte is for the Opcode and the other one is for the byte that stores the size of the key.

If these conditions satisfy then it returns `true`.

### `MultiSig`

Multi-signature scripts set a condition where N public keys are recorded in the script and at least M of those must provide signatures to release the encumbrance. This is also known as an M-of-N scheme, where N is the total number of keys and M is the threshold of signatures required for validation.

| Script | Contains |
| --- | --- |
| **Locking Script** | Specifies M and N and has N public key |
| **Unlocking Script** | M corresponding signatures |

A locking script setting an M-of-N multi-signature condition looks like this:

```
M <Public Key 1> <Public Key 2> ... <Public Key N> N OP_CHECKMULTISIG

```

Corresponding Unlocking Script for with M signatures will be:

```
OP_0 <Signature B> <Signature C> ...

```

> **P.S** The prefix `OP_0` is required because of a bug in the original implementation of `CHECKMULTISIG` where one item too many is popped off the stack. It is ignored by `CHECKMULTISIG` and is simply a placeholder.

The `Solver` function further calls `MatchMultisig`. It also passes two variables, one is `required` which stores the minimum number of signatures needed, basically the `m` value. The other is `keys` which store the n public keys, which will then be pushed to `vSolutions`.  
![multisig](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/multisig.png)

Let's dive into the `MatchMultisig`. It is a bit complicated because it doesn't have any fixed size. But one thing is for sure it must have the `OP_CHECKMULTISIG` at the end. And we can see that part is considered.

![MatchMultisig](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/MatchMultisig.png)

[src/script/standard.cpp#L124-L142](https://github.com/bitcoin/bitcoin/blob/71797beec54d36a055d5e172ecbf2141fa984428/src/script/standard.cpp?ref=blog.summerofbitcoin.org#L124-L142)

Let's move on to our final script format and a very interesting addition to the Bitcoin Core.

### `Pay-to-Script-Hash (P2SH)`

Was introduced to Bitcoin core in 2012 to resolve practical difficulties and to make the use of complex scripts as easy as a payment to a bitcoin address. Let's take the example of Alice and Bob. Let's assume Bob has a Multisig walltet (2-of-5) and Alice has to send some BTCs to Bob. Alice should include this in her output transaction.

```
2 PubKey1 PubKey2 PubKey3 PubKey4 PubKey5 5 OP_CHECKMULTISIG

```

But this is fairly complicated. But P2SH solves this issue.

With P2SH payments, the complex locking script is replaced with its digital fingerprint, a cryptographic hash. When a transaction attempting to spend the UTXO is presented later, it must contain the script that matches the hash, in addition to the unlocking script. In simple terms, P2SH means “pay to a script matching this hash, a script that will be presented later when this output is spent.”

| Script | Contains |
| --- | --- |
| **Redeem Script** | The actual logic of the transaction |
| **Locking Script** | Redeem script hash |
| **Unlocking Script** | In case of Multisig it will have the M signatures and the redeem script |

Redeem Script that Bob will keep with himself

```
2 PubKey1 PubKey2 PubKey3 PubKey4 PubKey5 5 OP_CHECKMULTISIG

```

Locking Script that Alice has to provide, you can see this is very simple and very similar to P2PK

```
OP_HASH160 20 <20-byte hash of redeem script> OP_EQUAL

```

Unlocking Script that Bob will use to redeem those coins by Alice

```
Sig1 Sig2 <redeem script>

```

One more advantage of using P2SH is that now Alice has no idea that Bob is using a Multisig. Here is how in the Solver function P2SH scripts are checked.

![p2sh](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/p2sh.png)

It calls a method `IsPayToScriptHash`. Now before looking at the method directly let's try to guess it's implementation.

The P2SH script looks like this:

```
OP_HASH160 20 <20-byte hash> OP_EQUAL

```

There are two opcodes so 2 bytes. 1 byte for the size i.e 20. And 20 bytes for the hash. In total it makes 23 bytes.  
Also, it should have`OP_HASH160` at the start and `OP_EQUAL` at the end. Let's look at the actual implementation:

![isp2sh](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/isp2sh.png)

Yup, that's exactly what we expected. Checks the size is 23, if the first opcode is `OP_HASH160` and then it states the size of the key which is 20 (0x14 is hex for 20) and then finally at the end we have `OP_EQUAL`.

This brings us to the end of the different TxoutTypes.

Let's head back to the `isStandard` function.

## IsStandard

![isstandard](https://raw.githubusercontent.com/arnabsen1729/sob-blogs/master/digging-isStandard/img/isstandard-1.png)

At this point, the `Solver` function has returned the type of the script. If by any chance the solver function wasn't able to decide on the standard, it returns `NONSTANDARD`.

If the `whichType` is of `NONSTANDARD`, it straight away returns `false`. If it has `MULTISIG` it does one final validation.

Remember we said `vSolutions` stores the important data, in the case of `MatchMultisig` it stores the m, the keys, and then n. So, for a multisig to be valid, n cannot be less than 1 and greater than 3 (as of 15 Aug 2021, Bitcoin master supports only x-of-3 multisig). Also, m cannot be less than 1 and greater than n.

Finally, there is another check for `NULL_DATA` type transaction. With the help of a small amount of transaction fee you can write something into the Bitcoin blockchain that will persist permanently, these transactions are called `NULL_DATA`, as they don't contain any transaction-related details. They are known as OP\_RETURN or Data Carrier txns too. These are considered nonstandard because these will never be used in future transactions.

If all these validations pass the `isStandard` function will return `true` and our script is not considered a standard transaction. All we have to do is to wait for it to be considered in a block.

---

I was a part of the [Summer of Bitcoin'21](https://summerofbitcoin.org/?ref=blog.summerofbitcoin.org) at the time of writing this blog. I am grateful to [Adi Shankara](https://twitter.com/adibitcoin?ref=blog.summerofbitcoin.org), [Caralie Chrisco](https://twitter.com/Caralie_C?ref=blog.summerofbitcoin.org), [Adam Jonas](https://twitter.com/adamcjonas?ref=blog.summerofbitcoin.org) for giving me this amazing opportunity and my mentor [0xb10c](https://twitter.com/0xB10C?ref=blog.summerofbitcoin.org) for his guidance and support.

If you want to get started with Bitcoin development checkout [the Summer of Bitcoin Resources](https://summerofbitcoin.org/?ref=blog.summerofbitcoin.org#resources)