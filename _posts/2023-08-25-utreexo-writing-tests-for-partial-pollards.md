---
layout: post
title: "Utreexo: Writing tests for Partial Pollards"
date: 2023-08-25
author: Aniket Shaha
categories: ['Tutorials', 'Utreexo']
---


Hello everyone, I have been working on the amazing open-source organization —
Utreexo for the Summer of Bitcoin 2023, being mentored by Calvin Kim, who has
been exceptional and patient in helping me get the intricacies of this
incredible project.

> What is Utreexo?

Utreexo is a hash accumulator for Bitcoin that was proposed by Tadge Dryja,
the co-author of the Lightning Network paper. It is built on the data
structure Merkle trees.

The current testing procedures within Utreexo are tailored to evaluate the
functionality of the full Pollard structure exclusively. However, it has
become evident that these tests fall short when applied to partial Pollard
structures. This discrepancy emphasizes the need for a more comprehensive
testing approach.

To address this, I have been diligently working on **developing tests** that
encompass partial Pollard structures. Partial pollards are the structure where
there is no need to remember all the nodes of every Bitcoin transaction but we
are only concerned with the nodes that are relevant to us. This helps a great
deal in compressing Bitcoin transactions to enable us to use it more
conveniently. These tests aim to bridge the gap and ensure that Utreexo’s
functionalities can seamlessly extend to these variations.

Ultimately, this endeavor contributes to the assurance that Utreexo remains a
dependable and versatile solution capable of optimizing blockchain storage
across diverse scenarios.

# The two important tests adopted:

  1. Testing the correctness of the cached nodes for the target nodes in Partial pollard to ensure that the nodes we are “caching” or “remembering” are actually valid and correct nodes to verify the existence of the target nodes.
  2. Testing that when we delete a target node in a Partial pollard, the nodes that are supposed to be remembered to prove the target node are “un-cached” correctly and enabling that other target nodes might have this node as a proof node for its existence. So, we do not remove the proof node entirely when we delete a target node.

These table-driven tests are designed to thoroughly evaluate the functionality
and reliability of a key component within the Utreexo project, which aims to
optimize blockchain storage. This component is referred to as the
“accumulator.” The accumulator’s role is to maintain and manage a dynamic set
of data elements (referred to as “nodes” or “leaves”) and efficiently provide
proof for the existence of specific nodes within this set.

> What these tests achieve and how are they actually implemented

  1. Test case specification

  * The tests are organized as a collection of structured test cases.
  * Each test case is defined by its name, the number of nodes to be added to the accumulator (accumulator leaves), and a list of indices representing nodes to be remembered (known as “remembered indices”).

2\. Test execution

  * For each test case, a new accumulator instance is initialized.
  * The specified number of nodes is added to the accumulator. Nodes marked with remembered indices are flagged for special treatment.

3\. Node Proving and Retrieval

  * For each remembered index, the corresponding node is retrieved from the accumulator. This validates that the accumulator can correctly access stored nodes.
  * A set of proof nodes required to verify the existence of remembered nodes is calculated.
  * The presence and retrievability of these proof nodes are tested, ensuring the accumulator’s capability to support proof generation.

4\. Proof Generation and Verification

  * The accumulator’s `Prove` function is utilized to construct proofs for the remembered nodes.
  * The generated proofs are validated using the `Verify` function, comparing them against the original remembered hashes. Successful verification confirms the accuracy of proof construction and verification.

5\. Proof nodes’ existence in the case of node deletion

  * After the deletion of some target nodes, it is ensured that the proof nodes of other target nodes that still need to be remembered and kept in the accumulator are safe and have not been deleted since they are needed further by these other target nodes.

6\. Outcome and Error correction

  * Any encountered errors or failures during the test are reported.
  * Successful completion of all test steps and verifications results in a report indicating the successful verification of the targeted nodes.

7\. Iteration over Test Cases

  * The entire process is repeated for each defined test case in a table-driven test approach, encompassing various combinations of node counts, rememberIndices, and delIndices.

# **Understanding the Utreexo tests in layman terms**

Imagine you have a big puzzle, and you want to prove that some puzzle pieces
are part of it without showing the entire puzzle. These tests are like
checking if you can do that correctly.

Here’s what these tests do in simple terms:

  1. They try different cases, like proving that one, two, three, and so on, pieces of the puzzle are in place.
  2. They check if the technology can prove the pieces are in the puzzle correctly.
  3. They see if the technology can also show other pieces (that are connected to the ones we’re proving) correctly.
  4. They make sure the technology can create and check special proofs that show the pieces we remember are in the right place.
  5. They repeat these checks for different situations, making sure the technology is good at its job.

These tests are sanity checks to ensure that partial Pollard structures also
work. Hope you learned a little about the world of Utreexo.

See you next time!
