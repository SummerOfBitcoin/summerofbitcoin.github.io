---
layout: post
title: A historic analysis of the topology of the Lightning Network 
date: 2025-10-14
image: https://ln-history.info/assets/writings/nx-top-10-nodes-rank.png
sortOrder: 3
author: Fabian Kraus
language: en
categories: ["Lightning-Network", "Development", "Open-Source", "Stories"]
---

> How did the topology of the Bitcoin Lightning Network change? 

> Who runs the nodes and who dominates the network?

---

# A historic analysis of the topology of the Lightning Network 

The Bitcoin Lightning Network consists of nodes and channels. Each channel has a capacity in `sats` which equals the amount of bitcoin locked.
Nodes publish `channel_announcement` messages to the Lightning Network. Other nodes recieve those messages and construct their own topology of the network.
When facilitating a payment a node needs to calculate the path of the payment. This step requires the node to know the topology of the network.

The Bitcoin Lightning network is a distributed network meaning that there is not a single source of trust, but every node has their own subjective view
The protocol has introduced three types of so called *gossip messages* that nodes can publish to let other peers know of updates of the topology. 

The [ln-history platform](https://ln-history.info) has collected those gossip messages and persists them in an index database. Utilizing the platform historic snapshots of the Lightning Network can be optained very quickly.

## Node Count Over Time

![Node Count Over Time](https://ln-history.info/assets/writings/nx-nodes.png)

The node count has grown rapidly during **2021** and **2022**. In the following years many node operators have decided to shut down their nodes. In **2025** we can see that the node count stays above **10000**.

## Network Capacity Over Time

![Network Capacity Over Time](https://ln-history.info/assets/writings/nx-locked-btc-w-fiat.png)

The number of locked bitcoin on the other hand has not seen a big drop as the node count. When comparing the locked bitcoin count with the equivalent value in USD the total capacity of the Lightning Network is at an all time high.
Taking into consideration that prices are denominated in USD, the Lightning Network has never held more liquidity (denominated in USD).


## Network Diameter Over Time
![Network Diameter Over Time](https://ln-history.info/assets/writings/nx-network-diameter.png)
When trying to see how the Lightning evolved we can take a look at the diameter of the network. The diameter of a graph is the longest shortest path. We can examine that this value has remained constant at **9** nodes. This means that the longest possible payment would need to take **9** hops.


## Median Payment Fee Over Time
![Median Payment Fee Over Time](https://ln-history.info/assets/writings/nx-median-payment-fee.png)

Nodes set up a fee policy for each outward channel direction. This information gets propagated using the `channel_update` message.
The fee policy of a channel consists of the `fee_base_msat` and the `fee_proportional_millionths`.
We simulate **10000** payments of **100000** `sats` and calculate the median fee of the payments. The fee is stable below **7** `sats`. 


## Altruistic nodes
![Share of nodes with fee base of zero](https://ln-history.info/assets/writings/nx-share-fee-base.png)

The graph shows that the share of channels with a `fee_base_msat` set to **0** is at slightly lower than **45 %**.
If we now check how many channels have set `fee_proportional_millionths` set to **0**, we get a value of around *8.5 %* of channels.

![Share of nodes with both fees set to zero](https://ln-history.info/assets/writings/nx-share-fee-proportional.png)

Combining both settings we can can see the following.
![Share of nodes with both fees set to zero](https://ln-history.info/assets/writings/nx-share-fee-base-and-fee-proportional.png)

This graph looks almost identical to the previous graph, indicating that if nodes set the `fee_proportional_millionths`to **0**, the `fee_base_msat` is also set to **0**. 

We can examine that the share of *altruistic nodes* is around **8.5 %**.


## Underlying Networks used

The Lightning Network can use both the clearnet with IP based routing as well as the more private Tor network with onion based routing.
![Share of nodes with both fees set to zero](https://ln-history.info/assets/writings/nx-share-node-address-type.png)

We can see that by just looking at the number of nodes, much more nodes run behind the Tor network. But this graph might be misleading as the next one shows.
![Share of nodes with both fees set to zero](https://ln-history.info/assets/writings/nx-share-node-address-type-capacity.png)

Most of the liquidity is managed by nodes using clearnet. The reason for that is that information speed is much higher in cleanet compared to Tor network.

## Top 10 nodes by channel count
![Top 10 nodes by channel count](https://ln-history.info/assets/writings/nx-top-10-nodes-rank.png)

Lastly we want to examine which nodes dominate the Lightning Network. A dominant node is a node with many channels.
We can see that enterprise nodes such as [ACINQ](https://acinq.co/), [1ML](https://1ml.com/), [WalletOfSatoshi](https://www.walletofsatoshi.com/) or [Kraken](https://www.kraken.com/) lead the list.


## Final words

Looking at the analyzed data it seems that the Lightning Network is maturing and professional. There is still a fair amount of node operators running and supporting the network altruistically. Nonetheless enterprises have the highest stake in the network.
