---
layout: post
title: "Summer of Bitcoin '23: Deep Dive into Utreexo"
date: 2023-09-10
author: Aniket Shaha
categories: ['Stories', 'Bitcoin Core']
---


Hello everyone, I have the amazing opportunity to contribute to open-source
bitcoin project through — _Utreexo : A Compact Data Structure for Bitcoin’s
UTXO Set_ , as part of my Summer of Bitcoin contribution.

_In this article, let us discuss further about what Utreexo data structure is
and its application in the Bitcoin network._

> **Introduction**

In the realm of blockchain technology, Bitcoin stands as the pioneering
cryptocurrency. Its underlying data structure, the Unspent Transaction Output
(UTXO) set, is crucial for verifying transaction inputs and maintaining the
integrity of the network. However, as the Bitcoin blockchain grows larger, the
UTXO set becomes increasingly tedious and inconvenient to store and process
efficiently.

To address this challenge, the project Utreexo was developed. It offers an
innovative data structure that significantly reduces the storage and
computational requirements of maintaining the UTXO set.

> **Understanding Utreexo**

Utreexo is a novel data structure representing a set of elements compactly. At
its core, Utreexo utilizes a group of Merkle trees, forming what is known as a
Merkle forest.

To integrate Utreexo into the existing Bitcoin network without requiring a
fork, a specific type of node known as a _Bridge Node_ is introduced. These
Bridge Nodes serve as intermediaries between the traditional UTXO set and the
Utreexo data structure. Their primary function is to convert the UTXO set into
the Utreexo forest, effectively reducing the storage burden.

The main goal behind Utreexo is to eliminate the need for storing all UTXOs,
instead organizing them into a condensed forest structure to reduce the
storage cost.

In a Utreexo tree, every node is the hash value of their children nodes. So,
if one knows the hash value of the two child nodes — we can generate the hash
value of the parent, and need not remember the parent node explicitly. This
goes right upto the root. We try to find the minimum number of nodes that are
needed in the tree for us to prove the existence of a given leaf node. This
significantly reduces transaction and storage costs!

> **Partial Trees**

While Bridge Nodes are instrumental in implementing Utreexo, the Utreexo
project aims to go one step ahead. An important feature of the project is
support of partial trees, as opposed to storing the entire forest within each
Bridge Node. By dividing the Utreexo forest among multiple Bridge Nodes, the
storage and computational requirements can be distributed, enabling efficient
and scalable operation.

> **The Four Utreexo Operations:**

The Utreexo data structure supports four fundamental operations: Add, Delete,
Prove, and Verify.

  1. Add: This operation involves adding a new element (a UTXO) to the Utreexo data structure. The Bridge Nodes incorporate the new UTXO into the appropriate Merkle tree within the forest.
  2. Delete: When a UTXO is spent, it must be removed from the Utreexo data structure. The corresponding Bridge Node identifies the relevant Merkle tree and eliminates the associated leaf node.
  3. Prove: Proving refers to generating a Merkle proof that verifies the inclusion of a particular UTXO within the Utreexo forest. This proof can be presented to other nodes for validation.
  4. Verify: The process of verification entails determining the validity of a Merkle proof. By performing a succession of cryptographic computations, Bridge Nodes and other network participants can validate the authenticity of the proof.

> **Summer of Bitcoin ’23 project:**

In order to improve the scalability of the Utreexo data structure, the Pollard
structure in the Utreexo library should be modified to support partial tree.
Currently, the library only supports storing all nodes, which limits its
scalability. By enabling partial tree functionality, we can compute larger
number of transactions.

  1. _Modification of the Pollard structure_ : The Utreexo library will undergo updates to support the storage of partial trees, allowing for more efficient use of resources.
  2. _Functionality of the four operations_ : The Add, Delete, Prove, and Verify operations will be modified to support partial trees and thoroughly tested to ensure that it works seamlessly within the Utreexo framework. This ensures that the Utreexo data structure can be effectively utilized in real-world scenarios.

I am excited to work in this project and contribute to an efficient future in
Bitcoin.
