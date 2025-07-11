---
layout: post
title: "Coin Selection for Dummies: Part 4 - Lowest Larger Selection"
author: Anmol Sharma
date: "2022-08-19 13:00:44 +0000"
categories:
  - "Tutorials"
  - "Coin Selection"
image: ../assets/images/blog_content/img-1.png
---

This is the fourth part of the seven-part series of blogs on Coin Selection.

* **Part 1:** [Overview of Coin Selection Algorithms](https://blog.summerofbitcoin.org/coin-selection-part-1/)
* **Part 2:** [Branch and Bound Coin Selection](https://blog.summerofbitcoin.org/coin-selection-for-dummies-2/)
* **Part 3:** [Knapsack Coin Selection](https://blog.summerofbitcoin.org/coin-selection-for-dummies-part-3/)
* **Part 4:** A new coin selection algorithm I proposed and implemented in Bcoin
* **Part 5:** Waste metric in Bitcoin
* **Part 6:** Comparison of different selection algorithms
* **Part 7:** How does fee estimation works?

If you haven’t checked out the first part, I suggest you read it before reading this blog because it explains some of the key concepts related to Coin Selection.

## **Recap**

In Computer Science, there are a large number of problems that belong to a class of NP-Hard Problem. In layman’s term, a NP-Hard problem is a type of problem which cannot be solved “***quickly***”. The time required to solve the problem grows exponentially with the size of inputs. For all geeks, a NP-Hard problem belongs to a class of problems that are informally "**at least as hard as the hardest problems in NP". 😉**

An example of an NP-hard problem is the **decision subset sum problem**. In the problem, we are given a set of integers and we need to find a non-empty subset that adds to zero.  
Another example of an NP-hard problem is the **optimization problem of finding the least-cost cyclic route** through all nodes of a weighted graph. This is commonly known as the **traveling salesman problem**.

## **What is the Binary Search Algorithm?**

Binary search is a search algorithm which is used to find the position of a target value within a **sorted array**. Binary search compares the middle of the element to the target, if they are not equal, the half in which the target cannot lie is eliminated and the search continues on the remaining half, again taking the middle element to compare to the target value, and repeating this until the target value is found.

Binary search is faster than linear search except in case of small array because binary search runs in logarithmic time in worst case, making O(log n) comparisons. Binary Search can also be used to find the next-largest or next-smallest element in the array relative to target.

### **Basic Steps to perform Binary Search:**

1. Begin search with the middle element as search key
2. If the value of the search key is equal to the target then return the index of search key
3. If the value of the serach key is less than target, eliminate the upper half
4. Otherwise, eliminate the upper half
5. Repeat until target is found

## **Lowest Larger Coin Selection**

We need to select a fixed number of UTXOs that sum up to a total amount while minimising fee. We can apply the concept of Binary Search to Coin Selection and select the coin which is closest to target.

## **Mechanics of Lowest Larger Selection Algorithm**

To use Lowest larger selection, we first sort UTXOs in decreasing order by value. For Binary Search to yield a valid result, we need to make sure the target is less than largest unselected UTXO. In order to do so, we keep selecting the largest unselected UTXO and decrease it’s value from the target till the target becomes smaller than the largest unselected UTXO. We can now perform binary search to find the UTXO whose value is equal to or greater than the target.

## **Implementation**

This is a simple Python Implementation of the algorithm.

```python
def selectLowestLarger(coins, target):
    coins.sort(key=lambda x: x.value, reverse=True)
    ind = 0
    selected = []

    # while target is greater than
    # the largest coin we have, we
    # will keep selecting the largest coin
    while target >= coins[ind].value:
        selected.append(coins[ind])
        target -= coins[ind].value
        ind += 1

    # we have reached the target, return selected
    if target == 0:
        return selected

    # now we are sure that
    # target < largest unselected coin
    # we will perform Binary search to
    # find the coin which is closest to target value
    lowestLarger = findLowestLarger(coins, ind, target)
    selected.append(lowestLarger)

    return selected

def findLowestLarger(coins, ind, target):
    i = ind
    j = len(coins) - 1

    # begin Binary Search
    lowestLarger = None
    while i <= j:
        mid = (i + j) // 2

        # if target is less than coin at middle
        # then search n right part of array
        if target <= coins[mid].value:
            lowestLarger = coins[mid]
            # repeat for right
            i = mid + 1
        else:
            # repeat for left
            j = mid - 1

    return lowestLarger

class Coin():
    def __init__(self, value):
        self.value = value

coins = [Coin(1), Coin(10), Coin(20), Coin(50), Coin(80), Coin(99), Coin(100)]

targets = [100, 121, 200, 11, 12, 30, 80, 300]

for t in targets:
    print()
    print("target: " + str(t))
    input = 0
    res = selectLowestLarger(coins, t)
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

### **Advantages**

* Extremely fast in finding the solution because it uses binary search
* Produces a small UTXO pool

### **Disadvantages**

* Sometimes produces a small change output which is not very useful for spending in future.
* Almost always produces a change output

In the next blog, we will learn about **waste metric** which is used to compare different selections and choose the best possible solution based on feerate.
