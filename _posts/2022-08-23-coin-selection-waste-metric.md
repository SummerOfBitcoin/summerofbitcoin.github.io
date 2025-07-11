---
layout: post
title: "Coin Selection for Dummies: Part 5 - Waste Metric"
author: Anmol Sharma
date: "2022-08-23 01:54:11 +0000"
categories:
  - "Tutorials"
  - "Coin Selection"
image: ../assets/images/blog_content/img-1.png
---

This is the fifth part of the seven-part series of blogs on Coin Selection.

* **Part 1:** [Overview of Coin Selection Algorithms](https://blog.summerofbitcoin.org/coin-selection-part-1/)
* **Part 2:** [Branch and Bound Coin Selection](https://blog.summerofbitcoin.org/coin-selection-for-dummies-2/)
* **Part 3:** [Knapsack Coin Selection](https://blog.summerofbitcoin.org/coin-selection-for-dummies-part-3/)
* **Part 4:** [A new coin selection algorithm I proposed and implemented in Bcoin](https://blog.summerofbitcoin.org/coin-selection-for-dummies-part-4-lowest-larger-selection/)
* **Part 5:** Waste metric in Bitcoin
* **Part 6:** Comparison of different selection algorithms
* **Part 7:** How does fee estimation works?

If you haven’t checked out the first part, I suggest you read it before reading this blog because it explains some of the key concepts related to Coin Selection.

## **Recap**

### **What is a UTXO?**

UTXO or Unspent Transaction Output is a technical term for a coin. A UTXO is an output that is generated as a result of a transaction. Every transaction in Bitcoin has some **inputs** (coins being spent or “***destroyed***”) and **outputs** (new coins being “***created***”).

### **What is Coin Selection?**

Coin Selection refers to the decision process of choosing **UTXOs** (or “*coins*”) to use as inputs when making an on-chain bitcoin payment. A wallet can associate the private keys with their respective UTXOs to create a UTXO pool. A UTXO pool consists of all the UTXOs or coins that a user can spend. The process of selecting UTXOs from this UTXO pool is called Coin Selection.

## **What is waste metric?**

Waste metric is heuristic used in Bitcoin Core’s wallet and now also in Bcoin to compare different possible input sets and select the set which produces the smallest waste score. Often there are multiple ways to fund a transaction, waste metric finds the most optimal input set based on fee rate, cost of producing a change and excess selection amount.

The most beautiful aspect of waste metric is that it also takes into account the cost of spending inputs now as opposed to sometime in the future. This is achieved by comparing current feerate to a long-term fee rate. In low feerate situations we would like to spend more inputs whereas in high feerate situations we would prefer to spend less inputs.

## **Why do we need waste metric?**

Often there are more than one way to select coins even in a small UTXO pool. For example, if our UTXO pool is [200, 300, 500, 1000] and we are trying to spend 500, we have the following selections:

1. Select **[200, 300]** - No change output
2. Select **[500]** - No change output
3. Select **[1000]** - Change output of **500**

We need a way to measure the effectiveness of these selections and use the best one out of these. We wouldn’t want to select **[1000]** because it unnecessarily generates a change when we have a changeless solution. We would prefer selecting **[200, 300]** in low feerate environments and **[500]** in high feerate environments.

Therefore, the problem of finding the best selection out of all possible selections is solved by waste metric.

## **Mechanics of Waste Metric**

The following factors are considered when calculating waste:

1. **Cost of change** includes the fees paid on this transaction’s change output plus the fees that will be paid to spend this change in future. If there is no change output, the cost is 0. The fee which will be be paid in future is calculated using long-term feerate.
2. **Excess selection amount** refers to the difference between the sum of selected inputs and the amount we need to pay (the sum of output values and fees). There shouldn’t be any excess if there is a change output.
3. **The cost of spending inputs now** is the fee difference between spending our selected inputs at the current fee rate and spending them later at the “long-term fee rate.” This helps us implement a long-term fee-minimization strategy by spending fewer inputs in high fee rate times and consolidating during low fee rate times.

Therefore the formula for waste comes out to be

`waste = selectionTotal - target + inputs × (currentFeeRate - longTermFeeRate)`

## **Implementation**

This is a simple Python Implementation of waste metric.

```python
LONG_TERM_FEERATE = 5 # sats/vB

def getWaste(coins, target, costOfChange, feeRate):
    waste = 0
    selectedAmount = 0

    for coin in coins:
        selectedAmount += coin.value
        # the cost of spending an input now vs in the future
        waste += (feeRate - LONG_TERM_FEERATE) * coin.value

    if costOfChange > 0:
        # the cost of making change and spending it in the future
        waste += costOfChange
    else:
        # the excess we are throwing away to fees
        waste += selectedAmount - target

    return waste

class Coin():
    def __init__(self, value):
        self.value = value

```

In the next blog, we’ll talk about the pros and cons of all the selection algorithms we’ve talked so far. Stay tuned!
