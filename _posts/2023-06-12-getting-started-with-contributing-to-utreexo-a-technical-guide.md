---
layout: post
title: "Getting Started with Contributing to Utreexo: A Technical Guide"
date: 2023-06-12
author: Aniket Shaha
categories: ['Stories', 'Utreexo']
---

I had the amazing opportunity to contribute to the Utreexo project throughout
the summer of 2023. Utreexo is an exciting open-source project that aims to
improve the scalability and efficiency of blockchain systems by using a novel
accumulator data structure — Merkle Trees. This article aims to guide you
through the process of getting started with contributing to Utreexo.

# Understanding Utreexo

Utreexo is a technology put forth by Thaddeus Dryja, aimed at minimizing the
data needed for validating Unspent Transaction Outputs (UTXOs) within a
blockchain. This is achieved through the utilization of a cryptographic
accumulator, which acts as a representation of a collection of UTXOs. This
enables the creation of proofs for confirming both the presence and absence of
items.

The primary goal of Utreexo is to reduce the storage and bandwidth
requirements for running a full node. In doing so, it has the potential to
enhance the scalability and decentralized nature of a blockchain network. In
Utreexo, a full node can keep only one hash per block, whereas for a
traditional pruned full node, you have to keep all the UTXOs per block. This
helps in reducing the storage significantly. The size of Utreexo can fit into
a QR code.

With a full node, a Bitcoin, all the UTXOs in existence will amount to upwards
of 4GB, but with the introduction of Utreexo, this has been reduced to a few
kilobytes!

# Technical Prerequisites

Before you begin contributing to Utreexo, it is recommended to be thorough
with the following prerequisites:

  1. **Blockchain Basics:** You should have a good understanding of blockchain technology, cryptographic concepts, and programming languages like Go.
  2. **Git and UNIX** : Familiarity with version control systems like Git and platforms like GitHub is essential for collaborating with the Utreexo community. Additionally, it is recommended to use Unix-like software and be familiar with the same.
  3. **Development Environment:** Set up a development environment on your computer with the Go programming language installed. You can download Go from the official website. (install go) You will need a version of `go 1.18` or newer.

# Contributing steps

  1. Fork or Clone the project to your local machine.
  2. **For fork** : On the Utreexo repository’s GitHub page, click the “Fork” button to create your own fork of the repository. This will allow you to work on your changes without affecting the main repository. After forking, clone the repository through the `HTTP` link in your forked GitHub repository.
  3. **For clone** : you can use **one** of the following commands: `go get github.com/utreexo/utreexo` or `git clone https://github.com/utreexo/utreexo`
  4. Explore the codebase to understand its structure, modules, and how Utreexo implements its accumulator and other components.
  5. accumulator_interface.go is the file that contains code to
  6. Run `go install` to install the project dependencies.
  7. **Create a new branch:** For each contribution, create a new branch. This helps keep your changes organized and makes it easier to track and merge them later.

# A brief explanation of the important files in the Utreexo codebase:

`pollard.go` contains code for a data structure called a "Pollard tree", which
is a binary tree used for efficient storage and retrieval of key-value pairs.
The file defines the `polNode` struct, which represents a node in the tree,
and provides functions for creating, inserting, and serializing/deserializing
Pollard trees.

`prove.go` contains code for a data structure called a "Proof", which is an
inclusion-proof for multiple leaves in a Utreexo tree. The file has the
definition for the `Proof` struct, which contains a list of leaf locations to
delete and all the nodes in the tree that are needed to hash up to the root of
the tree.

`polnode.go` contains code for the `polNode` struct, which represents a node
in a Pollard tree. The file defines functions for creating, inserting, and
deleting nodes in the tree, as well as functions for navigating the tree and
calculating the position of a node in the tree.

Along with these files, there are test files named `<filename>_test.go` that
test the functionalities of each file respectively.

I learned a lot by contributing to the Utreexo project which has a lot of
potential in scaling and growing bitcoin. Hope we see Utreexo having a direct
impact on greater Bitcoin acceptance.
