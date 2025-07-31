---
layout: post
title: Segment Trees in Lightning networks!
date: 2025-07-10
author: Abhijit Saha
categories: [Lightning Network]
image: ../assets/images/blog_content/2025-07-10-segment-trees-in-lightning-networks_bc715931.png
---

I have always been fascinated by segment trees. But as someone who craves to
find real world applications of every tiny piece of knowledge I find, I didn’t
appreciate the optimizations that it actually makes.

Okay but what the heck is a segment tree and actually what the heck is a
Lightning network?

Well, a comprehensive look at both of these things would require two separate
articles, hence I present here a brief first principles approach for both of
these:

**Segment trees (If you already know this then please skip to the next
section…trust me I’m not that smart to tell you something new)**

Segment trees are data structures that are most potently used for **Range
queries.** Suppose you want to find for a lot of queries the value of some
operation done over a _range_ of data in a data structure(most commonly
arrays).

Now if one wants to do that naively, you can just do that range operation for
each element in the structure _for each query._ Now, if that seems like a non-
optimal way, it is.__ Instead we come up with a beautiful way to sort of
**group** mini queries together such that those mini queries have a kind of
precomputed answer that we can utilize to more quickly calculate the main
query.

So suppose you have an array that consists of values that you want to find
range sums for (lets say you are given queries like find the sum from index 2
to 5, 4 to 6, 1 to 4 etc). You can consider each element to be kind of like a
leaf node, and then compute (as you go up the tree), pairwise sums (so the
next level would have nodes which represent the sum of 1 and 2, 3 and 4, and
so on), then the level above would have pairwise sums of these (like sum of
(el1+el2) and (el3+el4)).

source: [https://medium.com/hackernoon/practical-data-structures-for-frontend-
applications-when-to-use-segment-trees-9c7cdb7c2819](/hackernoon/practical-
data-structures-for-frontend-applications-when-to-use-segment-
trees-9c7cdb7c2819)

Now for each query, instead of going to the deepest level and computing the
sum element by element, we can just stop at a level which is completely inside
our queries (so for query 1–4, since 1–2 and 3–4 are completely inside our
range, we can just directly use their sum, instead of going down the level and
adding arr[3] and arr[4] to the answer). The crucial observation is that there
is at max 2 nodes at each level that are selected to compute the answer for a
particular query (you can check by taking some random examples). This means
that at max we would be needing 2*(No. of Levels) nodes to compute the answer.
This is the main observation that we are also going to be using in lightning
networks.

The main optimization comes when we need to change some values in the array.
For that, we would need to update the array, and then calculate the sum for
that query(if we use the naive approach), which is O(N) for a particular query
(too expensive). So instead, we could utilize another property of segment
trees (or trees in general), that binary trees have a height of log(N), where
N is the number of nodes.

How does that help? Well for a query that requires you to update a number, you
can change that index, and then propagate that change to its parent, and then
to its parent (siblings don’t matter). That way, since direct route from the
child to the parent is of log(N) length, you only need to do log(N) changes.
And since we would need 2*(No. of. Levels) = 2*Log(N) to actually compute the
answer, we can do the whole operation is O(log(N)) complexity. Great Success!

**Lightning Networks**

The lightning network is a protocol built on top of blockchain. It solves the
problem traditional blockchain faces which is the verification of transactions
taking too long.

Avoiding going into too much detail, lightning network uses HTLCs or Hashed
Time Locked Contracts. These are contracts/transactions that can be spent
after a specific amount of time by the party that broadcasts it (hence time-
locked).

The essence of it is opening and closing of channels. So if party A wants to
pay party B, they will agree to open a channel so that they can have
transactions that can fund either of the parties. Now suppose there is a
transaction by party A that gives party A 1 BTC (from its own wallet) and
transfers 9 BTC to party B. In the lightning network if such a transaction is
sent to party B after signing, then it is considered pretty much a valid
transaction, that requires no on chain verification (skipping a lot of details
here). Essentially, now if party B wants to pay party A (lets say 4 BTC),
they’ll have to revoke the previous transaction. Why? Well in a channel, if
you settle a transaction, then the channel is closed. So if for example after
sending 4 BTC to A, all of a sudden B decides to settle the transaction at the
first state (to his benefit), it is a loss for A. To prevent that, both
parties mutually agree on revealing a “Revocation key” for state N that
prevents them from broadcasting a previous state (that might be more
beneficial for them). How exactly?

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*KvrIBDgGtGpsK3jZ-jzXEA.png"/>
</figure>

Deleting and Adding states (source:
<https://ocw.mit.edu/courses/mas-s62-cryptocurrency-engineering-and-design-
spring-2018/resources/lec13-payment-channels-and-lightning-network/>)

As earlier said, broadcasting a transaction takes some fixed time to go
through, and it is this design choice that allows the other party to use the
data available in the broadcasted transaction + the revocation key provided by
the other party to essentially provide enough data to keep all the funds to
themselves! That is the a clever design choice that allows each party to hold
their horses…basically.

Wait wasn’t this about segment trees also? Okay so remember I told you A and B
need to mutually give each other their revocation key…well how do you optimize
their storage? Suppose you have got N revocation keys for N states, and A just
betrayed you and broadcasted the 1st state. You need to store all the N
revocation keys (O(N) in _space co_ mplexity) and search on them in log(N)
time or you can store a chain of hashes (kind of like blockchain itself) to
store only the final key, and kind of hash it continuously N number of times,
which is O(N) in _time_ complexity.

The solution? We utilize the unique property of segment trees, and how a node
can store the data of its children.

So the sender has this root which is a random string of characters. And from
it going left they append 0 to it and hash it to get the left child, and
append 1 to it and hash to to get the right child.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*S0KsolpDbl4fKoV4GBJ10Q.jpeg"/>
</figure>

source: the same video I linked in the previous image (really its a very
useful video if you want to learn more)

Assuming we are the receiver, we don’t know anything yet. So if we get
revocation key 1 and 2, we store them as leaf nodes. But if we get the key 3,
then we can say: well as per the data structure that we are considering, 3
should be the parent of 1 and 2, so we only store 3. And if we want to get the
value of keys 1 and 2, we can just append 0 and 1 respectively and hash to get
them.

This makes use of a property that I discussed earlier as well of there being
no more than 2 nodes that are required at each level. That holds here as well!
So we have optimized our storage to log(N), and since to search a particular
key we need to go at max from the root to the leaf, we have optimized the
search to log(N) as well. Log(N) space and Log(N) time complexity, much better
than the previous case.

This method is coined Elkrem by [Tadge
Dryja](https://ocw.mit.edu/search/?q=Tadge+Dryja), a sort of play of words of
Merkle (the reverse of Merkle root algorithm, if you know about that from
traditional blockchain).

Thus we conclude with an optimized method of storing revocation keys in Hashed
Time Locked Contracts in Lightning Network, if you ever want to flex that
piece of knowledge in front of peeps. Not that it is useless, I mean it is at
least better than millions who know about segment trees without seeing their
actual application.

If you have come this far, thank you for reading this and please let me know
if you want some clarifications regarding this (not me asking for more medium
article ideas 😅)

