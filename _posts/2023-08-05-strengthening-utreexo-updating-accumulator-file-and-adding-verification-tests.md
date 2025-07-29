---
layout: post
title: "Strengthening Utreexo: Updating Accumulator File and Adding Verification Tests"
date: 2023-08-05
author: Aniket Shaha
categories: ['Stories', 'Bitcoin Core']
---

> **Introduction** :

Welcome to another exciting exploration of Utreexo, the compact data structure
designed to optimize the storage and processing of Bitcoin’s Unspent
Transaction Output (UTXO) set. In our previous articles, we discussed
Utreexo’s fundamentals and its potential impact on the Bitcoin network. Today,
we will delve into two crucial aspects of Utreexo’s development that I
contributed in my next weeks: Updating the accumulator file to store only
necessary nodes for verification and adding verification tests in Golang.

**Updating the Accumulator File:**

The accumulator file in the Utreexo codebase plays a pivotal role in
maintaining the necessary information for verifying a particular leaf node
within the Utreexo forest. To enhance Utreexo’s efficiency and reduce storage
requirements, the accumulator file will be updated to remember and store only
the nodes essential for verification.

The key component involved in this update is the calculateNewRoot function. By
modifying this function, we can optimize the process of determining the new
root of the Utreexo forest after an Add or Delete operation. The updated
function will ensure that only the nodes required to verify the inclusion of
specific leaf nodes are retained in the accumulator file, significantly
reducing storage costs and improving overall performance.

**Adding Verification Tests in Golang:**

To ensure the accuracy and reliability of the updated accumulator file, it is
vital to thoroughly test its functionality. Verification tests play a crucial
role in confirming that the accumulator correctly holds the proof of nodes
required to verify a particular leaf node.

The Utreexo project includes the addition of comprehensive verification tests
in Golang. These tests will validate the functionality of the updated
accumulator file, ensuring that it retains the necessary information for
efficient and secure verification.

These tests will cover a wide range of Utreexo operations, including Add,
Delete, Prove, and Verify, while focusing on the specific behaviour of the
updated accumulator file.

By designing and executing these verification tests, we can identify and
address any potential issues or vulnerabilities in the updated accumulator.
Robust testing is an essential aspect of developing reliable software, and it
plays a crucial role in ensuring the stability and integrity of Utreexo.

> **Conclusion:**

It is a great time building on the project by small but important steps that
helps me gain understanding of the project and also build a deeper sense of
amazement by Bitcoin everyday. I look forward to adding more test cases —
Ensuring that deleting a node also deletes its associated nodes that need to
remembered.

Happy coding and contributing!
