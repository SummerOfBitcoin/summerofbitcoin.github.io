---
layout: post
title: "Rust-Miniscript's Planning Module: Efficient Bitcoin Transactions"
date: 2023-08-01
author: Harshil Jani
categories: ['Stories', 'Rust']
---


Share

I have worked on the **Rust-Miniscript** project during my **Summer of Bitcoin
2023 Internship**. This blog post is about what actually is the Planning
Module, Why we need it and How it solves the problem in the Bitcoin World. We
will slowly transition in between over the work which I have done throughout
the internship to make improvements on the Planning module.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Fbg9e8bg2aiggO4OPZbVBA.png"/>
</figure>

Photo by [Michael
Förtsch](https://unsplash.com/@michael_f?utm_source=medium&utm_medium=referral)
on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

Before we get into the details of Planning module, I would like to add
information about all the parts of the projects and my understanding about
them. The article is divided into sections and you can navigate into the
sections which you are relating the most with. Reading all the sections should
help new commers into the Bitcoin Development give more clear insights. Let’s
get started.

# Bitcoin Scripts

Bitcoin Script is a simple, stack-based programming language used in the
Bitcoin protocol to define the spending conditions for Bitcoin transactions.
It is an integral part of how Bitcoin’s blockchain enforces security and
control over the movement of funds. Bitcoin Script allows users to specify the
rules that must be met in order to spend the funds locked in a particular
output (also known as a “UTXO” or unspent transaction output).

Each Bitcoin transaction includes one or more inputs and outputs. The input of
a transaction references a previous output (UTXO) and provides the necessary
data to satisfy the spending conditions defined by the output’s locking
script. The locking script, also known as the “scriptPubKey,” specifies the
rules that must be met for the funds to be spent. The input includes a
separate script called the “scriptSig,” which provides the data required to
satisfy the locking script’s conditions.

Bitcoin Script operations include simple mathematical operations, bitwise
operations, cryptographic operations, and conditional branching. It is
intentionally designed to be limited in complexity to ensure security and
prevent malicious or unintended behavior. This design choice is meant to
minimize the potential attack surface and prevent vulnerabilities that could
compromise the integrity of the Bitcoin network.

# PSBT

Partially Signed Bitcoin Transaction (PSBT) is a standardized format that
allows multiple parties to collaboratively build and sign a Bitcoin
transaction without exposing sensitive private keys. It’s designed to
facilitate offline signing, multi-signature setups, and hardware wallet
integrations, making the process of creating and signing Bitcoin transactions
more secure and flexible.

PSBT enables the separation of the transaction creation process from the
signing process, which is particularly useful for scenarios where parties want
to maintain the security of their private keys while collaborating on
transaction creation.

# **Miniscript**

Miniscript is a scripting language designed to make it easier to create and
analyze Bitcoin scripts. Traditional Bitcoin scripting can be complex and
difficult to reason about. Miniscript aims to simplify this process by
providing a more structured and human-readable way to express spending
conditions.

# Key features of Miniscript

** _Descriptors :_** Descriptors are used to define the spending conditions
for outputs in a more human-readable and abstract way, making it easier to
create and manage complex scripts.

**_Policy :_** The policy is a mechanism in Rust-Miniscript that determines
whether a descriptor can be satisfied or not. It involves evaluating whether
the given spending conditions have been met, and if so, allowing the spending
of the associated UTXO. This layer of abstraction further enhances the
readability and usability of Rust-Miniscript, making it accessible even to
those without an in-depth understanding of Bitcoin scripting.

**_Compiler :_** The compiler bridges the gap between the human-readable
descriptors and the machine-executable scripts. It takes the descriptor as
input and generates the corresponding Bitcoin script, optimizing opcode
selection and assembly. This step minimizes the potential for error and
ensures that the resulting scripts are efficient and secure.

# Fee Rate of a Bitcoin Transaction

Each bitcoin transaction is verified by a network of nodes and it requires a
transaction fee to process the transaction on blockchain.

In case of multi-party transactions, it is important to agree on a transaction
fee rate to prevent DOS attacks. To address this problem, rust-miniscript has
implemented the absolute worst-case analysis for the satisfaction of weights
and employs it to calculate the fees.

# Need for Planning Module

Since we already know we require the fee rate to be calculated. Now assume
that for multisig transactions you would need 2 of 3 keys and you are sure
enough that one of the key is not present for sure. Then you may want to avoid
calculating fee on that key. But as per worst case implementation for **k of
n** multisig transactions wallets were built such that by default n keys were
required which means

**satisfaction weight = n*weight of each key**

To solve this problem of over-spending in case of asset unavailability, The
planning module plays a vital role. It allows you to choose a spend path. So
you can now choose **k** keys only instead of all n. This helps in reducing
fee rates to **k*weight of each key. (k≤n)**

# What is Planning Module

A plan is represented by a particular spending path on a descriptor (rules for
spending transaction output). It is a way using which we can analyze a choice
of spending path without actually producing any signatures or other witness
data.

To create a spending plan, We need to provide a descriptor along with keys or
pre-image hashes and absolute and relative time lock constraints. Here we can
also get which subset of the available keys and pre image hashes we will
actually need to use. This is helpful for constructing a Bitcoin transaction.

Once we obtain the required signatures, hash pre images, etc. the spending
plan can create a **witness or script_sig** for the input. Which can be used
to sign the transaction and spend output defined by the descriptor.

Instead of building the **Witness** [ signatures, hash pre-images to verify
ownership] , the **Witness Templates** are being built, Which contain
placeholders instead of actual signature and preimage. This would help in
determining the weight of the witness more accurately before constructing the
actual witness.

The entire workflow using the Planner module should seem as shown below.

Workflow for planner module

<https://github.com/rust-bitcoin/rust-miniscript/pull/481>

# Example to understand Miniscript and Planning

One part of my work was to write example tutorials for the developers who
would need to use Rust-Miniscript as a crate into their wallet backend.
Inorder to explain the entire workflow about the Miniscript and the Planning
Module I am writing a sample example below.

**Explanation :** We are trying to write a policy which has

key A **AND** [key B **OR** key C **OR** key D]

**Spend Policy :** and(pk(A),or(pk(B),or(pk(C),pk(D))))

**Miniscript Output :** and_v(or_c(pk(B),vc:or_i(pk_h(C),pk_h(D))),pk(A))

**Bitcoin Script :**

    
    
     <B> OP_CHECKSIG OP_NOTIF  
      OP_IF  
        OP_DUP OP_HASH160 <HASH160(C)> OP_EQUALVERIFY  
      OP_ELSE  
        OP_DUP OP_HASH160 <HASH160(D)> OP_EQUALVERIFY  
      OP_ENDIF  
      OP_CHECKSIGVERIFY  
    OP_ENDIF  
    <A> OP_CHECKSIG

**Satisfaction Weight and Planning Module :**

Considering these keys as a part of **wsh** descriptor which uses the ECDSA
Signatures of length 72 Bytes. Satisfying this script would cost us
approximates 4*72 = 288 Bytes. But there might be other costs added for
pushing or dissatisfying the script.

The above is the case in Worst-Case Analysis.

Now with the planning module, We get to choose to select our path. For example
the possible paths in the above policy would be

  1. key A , key B (144 Bytes)
  2. key A , key C (144 Bytes)
  3. key A , key D (144 Bytes)
  4. key A , key B , key C (216 Bytes)
  5. key A , key B , key D (216 Bytes)
  6. key A , key C , key D (216 Bytes)
  7. key A , key B , key C , key D (288 Bytes)

It is obvious that we can select the keys with least number of assets which
here is combination of two keys which costs 144 bytes. And we can almost half
the fee rate by choosing that spend path and develop PSBT Input.

This is exactly what is possible with the help of planning module by choosing
a particular spend path.

As the work was in collaboration with BDK developers, We are waiting for the
#481 to get merged and my PR is built on top of their work. The below is the
PR which adds the APIs for obtaining all the possible planned spend path and
counting the assets in linear time to check for the DOS attacks possibilities.

> <https://github.com/rust-bitcoin/rust-miniscript/pull/559>

# Improvements in Planning Module

I have worked on the logics behind getting all the combination of plans for a
given descriptor and policy. So, In the above example, we have the 7 possible
spend paths. I wrote a method called **get_all_assets** which recursively
checks into the different miniscript elements and returns a vector of Assets
which can be built from the policy paths.

<https://github.com/rust-bitcoin/rust-miniscript/pull/559>

In order to be protected from DOS Attack we would need to check total number
of plans. If it exceeds a certain threshold, we will simply reject the
descriptor. This is important because in case if the algorithm tries to plan
the satisfaction for all possibilities it would end up increasing the
congestion over the network and we just don’t want this to happen.

We have two different functions for getting the assets and counting all the
assets because getting assets happens in an exponential complexity but simple
counting of assets should take less time if we can already compute the
combinomatric formulation for miniscript elements.

Eg : AND(A,B) would give only 1 solution [(key A, key B)]

OR(A,B) would give 2 solutions [(key A),(key B)]

multi (k,A1,A2,A3,…,An) would give nCk possibilities.

# Challenges

When miniscript tries to finalize the PSBT, it doesn’t have the full
descriptor (which contained a pkh() fragment) and instead resorts to parsing
the raw script sig, which is translated into a “expr_raw_pkh” internally. We
introduced the method **substitute_raw_pkh** to convert the expr_raw_pkh into
pkh by looking at the available keys and their hases being stored inside of
the Hashmap of PSBT Input. If the key is found corresponding to given hash, It
will update to pkh.

For ECDSA signatures in `BIP32 Derivations` we get all the keys included for a
given descriptor. And for Schorr Sigs in Taproot we get into
`tap_key_origins`. Here we need all the keys in order to provide a verdict
whether we want to satisfy them or dissatisfy them. Those keys are mapped with
their key hashes and we simply replace expr_raw_pkh values with the PKH(key)
to make it back into the non hashed form. This reverse translates the
descriptor and we can retain its property back and finalize it for
transaction.

<https://github.com/rust-bitcoin/rust-miniscript/pull/557>

<https://github.com/rust-bitcoin/rust-miniscript/pull/589>

# Conclusion

The planning module in Rust-Miniscript revolutionizes the way developers
create Bitcoin scripts by offering a high-level, human-readable approach to
defining spending conditions. This abstraction enhances security, readability,
and efficiency, making it an invaluable tool for crafting intricate spending
scenarios while minimizing the risk of scripting errors. Incorporating Rust-
Miniscript’s planning module into your projects can significantly streamline
the process of building complex Bitcoin transactions.

If you’re interested in maximizing the potential of Rust-Miniscript, exploring
the planning module is an essential step on your journey to becoming a
proficient Bitcoin script developer.

Please do consider contributing to this amazing project. You should not be
restricted by the limited Bitcoin Knowledge. If you know basics of programming
and bit of crux in rust language, You can always find some beginner friendly
issues and make them look good for the project. 😉


