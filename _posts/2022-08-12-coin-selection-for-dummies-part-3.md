---
layout: post
title: "Coin Selection for Dummies: Part 3 - Knapsack Solver"
author: Summer of Bitcoin
date: "2022-08-12 10:46:47 +0000"
tags:
  - "Tutorials"
---

********By Anmol Sharma********  
********Summer of Bitcoin '22********

This is the third part of the seven-part series of blogs on Coin Selection.

* **Part 1:** [Overview of Coin Selection Algorithms](https://blog.summerofbitcoin.org/coin-selection-part-1/)
* **Part 2:** [Branch and Bound Coin Selection](https://blog.summerofbitcoin.org/coin-selection-for-dummies-2/)
* **Part 3:** Knapsack Coin Selection
* **Part 4:** A new coin selection algorithm I proposed and implemented in Bcoin
* **Part 5:** Waste metric in Bitcoin
* **Part 6:** Comparison of different selection algorithms
* **Part 7:** How does fee estimation works?

If you haven’t checked out the first part, I suggest you read it before reading this one because it explains some of the key concepts related to Coin Selection.

## **Recap**

In Computer Science, there are a large number of problems that belong to a class of NP-Hard Problem. In layman’s term, a NP-Hard problem is a type of problem which cannot be solved “***quickly***”. The time required to solve the problem grows exponentially with the size of inputs. For all geeks, a NP-Hard problem belongs to a class of problems that are informally "**at least as hard as the hardest problems in NP". 😉**

An example of an NP-hard problem is the **decision subset sum problem**. In the problem, we are given a set of integers and we need to find a non-empty subset that adds to zero.  
Another example of an NP-hard problem is the **optimization problem of finding the least-cost cyclic route** through all nodes of a weighted graph. This is commonly known as the **traveling salesman problem**.

## **What is the Knapsack Problem?**

The knapsack problem is a decision-making problem in which, we are given a set of items, each with a weight and a value. We need to determine the number of each item to include in a collection so that the total weight is less than or equal to a given limit and the total value is as large as possible. While the decision Knapsack problem is NP-complete, the optimization problem is not but its resolution is at least as difficult as the decision problem, and there is no known polynomial algorithm which can tell if a given solution is optimal.

## **Knapsack in Coin Selection**

The Knapsack problem can be applied to coin selection. We need to select a fixed number of UTXOs that sum up to a total amount while minimising fee. Since Knapsack problem is NP-complete, there is no known algorithm both correct and fast (polynomial-time) in all cases. Therefore, this causes Knapsack to take a large amount of computational resources

## **Mechanics of Knapsack Algorithm**

The Knapsack algorithm performs two runs. In first run, it tries to find the smallest excess over the given target (spending target + estimated fee). In the second run, the target is adjusted to generate a change target. Therefore, the target in second run is spending target + estimated fee + change. All of these runs are performed on UTXOs which are smaller than target because any UTXO greater than target will cause an overshoot.

In each run, solver performs upto 1000 iterations. The UTXOs are sorted in descending order by value. The solver selects these coins in two passes. In the first pass, each UTXO is selected randomly with a 50% chance. If the selected amount exceeds the target, then we have a solution. Current solution is compared to the best solution so far and stored as best solution if it’s smaller than best solution. The solver continues by deselecting the previously selected UTXO and continues the search. It attempts to replace the last selected UTXO with smaller UTXO(s) to produce even smaller sufficient set. If first pass produces a valid solution, then it is returned.

If the first pass didn’t produce any solution then, a second pass is performed in which every unselected UTXO is selected. Similar to first pass, after finding a solution, the search continues for a smaller solution.

## **Implementation**

This is a simple Python Implementation of the algorithm.

```
import random

# maximum number of iterations
ITERATIONS = 100000

def selectKnapsack(coins, target):
    # sort the coins in decreasing order of value
    coins.sort(key=lambda x: x.value, reverse=True)
    i = 0
    # select from values less than target
    while i < len(coins) and coins[i].value > target:
        i += 1
    # if no coins are less than target, return empty list
    if i == len(coins):
        return []
    else:
        coins = coins[i:]

    # initialize by selecting the all coins
    best = [True] * len(coins)
    nBest = 0
    for coin in coins:
        nBest += coin.value

    # if all coins are selected, and the sum is less than target
    # return empty list
    if nBest < target:
        return []

    iter = 0
    while iter < ITERATIONS and nBest != target:
        isIncluded = [False] * len(coins)
        reachedTarget = False
        nTotal = 0  # total value of selected coins
        # in pass == 0, select coins randomly
        # in pass == 1, select coins greedily
        for Pass in range(2):
            for i in range(len(coins)):
                if bool(random.getrandbits(1)) if Pass == 0 else not isIncluded[i]:
                    nTotal += coins[i].value
                    isIncluded[i] = True
                    if nTotal >= target:
                        reachedTarget = True
                        # if nTotal is between target and nBest, update nBest
                        if nTotal < nBest:
                            nBest = nTotal
                            best = isIncluded[:]
                        nTotal -= coins[i].value
                        isIncluded[i] = False
        iter += 1

    # return the selected coins
    return [coins[i] for i in range(len(coins)) if best[i]]

class Coin():
    def __init__(self, value):
        self.value = value

coins = [Coin(1), Coin(10), Coin(20), Coin(50), Coin(80), Coin(99), Coin(100)]

targets = [10, 15, 25, 55, 75, 100, 190, 250, 300, 350, 400]

for t in targets:
    print()
    print("target: " + str(t))
    input = 0
    res = selectKnapsack(coins, t)
    if len(res) == 0:
        print("No solution possible!!")
        continue
    print("coins: ", end="")
    for c in res:
        input += c.value
        print(c.value, end=" ")
    print()
    print("change: " + str(input - t))

```

### **Advantages**

* Improves privacy by selecting UTXOs pseudo-randomly that have no consistent fingerprints like age or value.
* Produces a small UTXO pool

### **Disadvantages**

* Aims to create a minimum change output of 10mBTC which reduces privacy
* Aggressively consolidates tiny UTXOs including uneconomical outputs
* Computationally expensive
* Likely to cause unnecessarily large transactions fees

In the next part, we will do a deep-dive into mechanics of Lowest Larger Selection Algorithm.

PS: The algorithm I'll talk about in the next part was proposed by me. Stay tuned!!