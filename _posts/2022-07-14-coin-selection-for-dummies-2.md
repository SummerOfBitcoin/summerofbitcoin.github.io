---
layout: post
title: "Coin Selection for Dummies: Part 2-Branch and Bound Coin Selection"
author: Anmol Sharma
date: "2022-07-14 08:42:17 +0000"
categories:
  - "Tutorials"
  - "Coin Selection"
image: ../assets/images/blog_content/img-1.png
---

This is the second part of the seven-part series of blogs on Coin Selection.

* **Part 1:** [Overview of Coin Selection Algorithms](https://blog.summerofbitcoin.org/coin-selection-part-1/)
* **Part 2:** Branch and Bound Coin Selection
* **Part 3:** Knapsack Coin Selection
* **Part 4:** A new coin selection algorithm I proposed and implemented in Bcoin
* **Part 5:** Waste metric in Bitcoin
* **Part 6:** Comparison of different selection algorithms
* **Part 7:** How does fee estimation works?

If you haven’t checked out the first part, I suggest you read it before reading this blog because it explains some of the key concepts related to Coin Selection.

## **Overview**

In Computer Science, there are a large number of problems that belong to a class of NP-Hard Problem. In layman’s term, a NP-Hard problem is a type of problem which cannot be solved “***quickly***”. The time required to solve the problem grows exponentially with the size of inputs. For all geeks, a NP-Hard problem belongs to a class of problems that are informally "**at least as hard as the hardest problems in NP". 😉**

An example of an NP-hard problem is the **decision subset sum problem**. In the problem, we are given a set of integers and we need to find a non-empty subset that adds to zero.  
Another example of an NP-hard problem is the **optimization problem of finding the least-cost cyclic route** through all nodes of a weighted graph. This is commonly known as the **traveling salesman problem**.

## **What is Branch and Bound Algorithm?**

Branch and Bound Algorithms are algorithms which are generally used for solving combinatorial optimization problems. These problems are generally classified as NP-Hard problems. These problems are typically exponential in terms of time complexity and may require exploring all possible permutations in worst case. Branch and bound (BnB) is an algorithm paradigm widely used for solving such problems. A branch and bound algorithm explores the entire search space of possible solutions and provides an optimal solution.

## **Branch and Bound in Coin Selection**

Branch and Bound can be used to solve coin selection problem because coin selection is a type of subset sum problem in which we need to find a subset which is equal to a given target. Branch and Bound can be used to systematically search the entire search space. Moreover, Branch and Bound finds an exact solution which does not produce a change output. Branch and Bound can sometimes take a large amount of computational resources and still produce no result. Therefore, it is necessary to limit the total number of iterations in Branch and Bound.

## **Mechanics of Branch and Bound**

In BnB, instead of repeatedly searching the same combinations, the combinations of UTXOs can be searched once with less effort. A binary decision tree is constructed where each level relates to the inclusion or omission of a UTXO.

One of the main ideas of Branch and Bound is to use **effective value of UTXOs**. When we create a transaction, we need to pay fee according to the size of the transaction. The size of transaction will increase if we use more inputs. This makes coin selection even harder. Adaptive fee estimation during the selection is an important feature as the selection results in valid results only. Since input scripts and output scripts have fixed data sizes, the cost per input and output are fixed known values within one payment. Thus we can calculate the effective value (the contribution of a UTXO towards a target).  
$**effectiveValue = UTXO.value - feePerByte \* bytesPerInput$**  
If we use effective value for selection the target will remain fixed because every UTXO will spend itself. Thus, target won’t increase even if we select more UTXOs.

**Depth-first search (DFS)** is used to explore the tree. UTXOs are sorted in decreasing order by their effective values and the tree is explored deterministically per the inclusion branch first. At each node, the algorithm checks whether the selection is within the target range or not. While the selection has not reached the target range, more UTXOs are included.

When a selection’s value exceeds the target range, the complete subtree deriving from this selection can be omitted. At that point, the last included UTXO is deleted and the corresponding omission branch is explored instead. The search ends after the complete tree has been selected or after a limited number of tries.

### **Understanding by example**

Let’s understand using an example:

For simplicity, we will assume fee to be zero.

**Coins = [100, 50, 20, 10, 1]** and **Target = 121**

The following decision tree will be constructed:

![img](../assets/images/blog_content/img.png)

1. We start by selecting the first coin **100.**
2. **Current value = 100** and **Target = 121**
3. We select the next coin of value **50.**
4. **Current value = 150** and **Target = 121**
5. Since **Current value > Target**, we will backtrack unselect **50.**
6. **Current value = 100** and **Target = 121**
7. We select the next coin of value **20.**
8. **Current value = 120** and **Target = 121**
9. We select the next coin of value **10.**
10. **Current value = 130** and **Target = 121**
11. Since **Current value > Target**, we will backtrack unselect **10.**
12. **Current value = 120** and **Target = 121**
13. We select the next coin of value **10.**
14. **Current value = 121** and **Target = 121**
15. We are done since!!

Thus our selection set is **[100, 20, 1]**

The search continues to find better solutions after one solution has been found. The best solution is chosen by minimizing the waste metric. (Refer to my previous blog of a brief understanding of waste metric, I’ll be publishing a deep-dive blog on Waste Metric in Week 5)

Bitcoin Core’s implementation of the algorithm also uses two additional optimizations. A lookahead keeps track of the total value of unexplored UTXOs. A subtree is not explored if the lookahead indicates that the target range cannot be reached. This allows for skipping unnecessary combinations.

## **Implementation**

This is a simple Python Implementation of the algorithm.

```python
TOTAL_TRIES = 100000

def selectBnB(coins, target):
    selected = []
    currValue = 0

    # calculate total available value
    totalValue = 0
    for coin in coins:
        totalValue += coin.value

    # we don't have enough balance
    if totalValue < target:
        return []

    # sort the coins
    coins.sort(key=lambda x: x.value, reverse=True)

    currTry = 0
    utxo_pool_index = 0

    # perform a depth-first search for choosing UTXOs
    while currTry < TOTAL_TRIES:
        backtrack = False
        # conditions for backtracking
        # 1. cannot reach target with remaining amount
        # 2. selected value is greater than upperbound
        if currValue + totalValue < target or currValue > target:
            backtrack = True
        # if selected value is equal to target, we are done
        elif currValue == target:
            break

        # backtrack if necessary
        if backtrack:
            # we walked back to first UTXO,
            # all branches are traversed, we are done
            if len(selected) == 0:
                break

            # Add omitted UTXOs back before traversing the omission branch of last included UTXO.
            utxo_pool_index -= 1
            while utxo_pool_index > selected[-1]:
                totalValue += coins[utxo_pool_index].value
                utxo_pool_index -= 1

            # Remove last included UTXO from selected list.
            currValue -= coins[utxo_pool_index].value
            selected.pop()
        # continue on this branch, add the next UTXO to selected list
        else:
            coin = coins[utxo_pool_index]
            # remove this UTXO from total available amount
            totalValue -= coin.value
            # if this UTXO is the first one or
            # if the previous index is included and therefore not relevant for exclusion shortcut or
            # if this UTXO's value is different from the previous one,
            if len(selected) == 0 or utxo_pool_index - 1 == selected[-1] or coin.value != coins[utxo_pool_index - 1].value:
                selected.append(utxo_pool_index)
                currValue += coins[utxo_pool_index].value

        currTry += 1
        utxo_pool_index += 1

    # if we exhausted all tries, return empty list
    if currTry >= TOTAL_TRIES:
        return []

    # return the selected UTXOs
    result = []
    for i in selected:
        result.append(coins[i])
    return result

class Coin():
    def __init__(self, value):
        self.value = value

coins = [Coin(1), Coin(10), Coin(20), Coin(50), Coin(80), Coin(99), Coin(100)]

targets = [100, 121, 200, 11, 12, 30, 80, 1000]

for t in targets:
    print()
    print("target: " + str(t))
    input = 0
    res = selectBnB(coins, t)
    if len(res) == 0:
        print("No exact solution possible!!")
        continue
    print("coins: ", end="")
    for c in res:
        input += c.value
        print(c.value, end=" ")
    print()
    print("change: " + str(input - t))

```

## **Advantages**

* Creates no change output which reduces current fees, future fees, cuts transaction graph for wallet, and has consolidatory effect on wallet's UTXO pool.
* Improves privacy by selecting UTXOs that have no consistent fingerprints like age or value.
* Uses minimal input set among viable candidates reducing address linkage and fees further improving privacy.
* Utilises waste metric to spend more inputs at lower feerates and fewer inputs at higher feerates.
* Prefers spending less blockspace efficient output types at lower feerates and more efficient output types at higher feerates due to waste metric.

## **Disadvantages**

* Does not always produce a solution. A changeless solution might not be possible.
* If a transaction is stuck, sender cannot CPFP (Child Pays for Parent) since there is no change output.
* It is more complicated to implement expensive to compute.

In the next blog, we will do a deep-dive into mechanics of Knapsack Coin Selection Algorithm. Stay tuned!
