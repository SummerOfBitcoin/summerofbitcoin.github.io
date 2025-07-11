---
layout: post
title: mempool.space
author: Priyansh Rastogi
date: "2022-01-17 15:19:00 +0000"
categories:
  - "Tutorials"
description: "Making smart decisions through mempool visualizer!"
image: https://cdn.hashnode.com/res/hashnode/image/upload/v1626617156416/TdAisKaaJ.png
---

In the years 2015-2017, it was really very easy to get bitcoin payments done with minimum fees, since the market was not very competitive.

But in the recent years, the bitcoin payments market has seen a lot of growth.  
Hence more congestion. With this the fee market evolved. 📈

![Screenshot from 2021-07-18 19-27-36.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626616689256/pUVgvEMLf.png)

> *Number of Blockchain wallet users worldwide from November 2011 to July 11, 2021*

Due to highly competitive market the fees grew exponentially.  
But guess what, you could save fees by making good decisions.  
But how do we do so? 🤔  
Analyze the mempool!

*mempool: a list of unconfirmed transactions in blockchain*

And this is where **mempool.space** emerges.

Uptil now you might have heard of bitcoin block explorer.  
Mempool.space is a bitcoin block and mempool explorer. 😎

A mempool explorer helps you to see the state of mempool and can be really helpful to estimate fees and transaction confirmation time.

![Screenshot from 2021-07-18 19-20-52.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626616267126/tH4FoG9Vi.png)

Let's dive deeper. 🌊

Here you can see a queue of confirmed and unconfirmed blocks.  
![Screenshot from 2021-07-18 19-35-24.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626617156416/TdAisKaaJ.png)

These are the unconfirmed ones. This is the visual representation of mempool we were talking about.  
![Screenshot from 2021-07-18 19-35-35.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626617281490/ZF0Zg6lb6.png)

Let's analyze an unconfirmed block.

![Screenshot from 2021-07-18 19-40-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626617416205/xrBawgWf5.png)

The transactions in this block have a median fee of ~2 sat/vB ranging from 1-605 sat/vB and currently it holds 1053 transactions.  
This block will most probably be mined in the next ~10 minutes.

The state of the block changes until the block is mined successfully. Miners look for transactions with max fee as newer and newer transactions get into the mempool.  
See, the median fee changes to 3 sat/vB.

![Screenshot from 2021-07-18 19-51-59.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626618279545/m_7G-t8Vq.png)

And there its mined.✔️ Beautiful!

![Screenshot from 2021-07-18 19-54-46.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626618309063/a81c2OVqkh.png)

Now let's see how to read a mempool and estimate fees for our transaction. 🧐

![Screenshot from 2021-07-18 21-23-52.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626623655104/J-tIWOojS.png)

Mempool.space suggests the fees based on the state of the current mempool.  
It also has graphs showing the accepted fee trends in past and live incoming transactions size.

The chart itself represents the current mempool. The colored layers represent transaction fee rates.

![Screenshot from 2021-07-18 21-30-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626625555365/yXLb-TOG7.png)

As you can see above, the pink at the bottom represents unconfirmed transactions in the mempool with a 1 - 2 sat/vB fee rate.

In the layer above that, you see unconfirmed transactions with a 2 - 3 sat/vB fee rate. The layer above that, 3 - 4 sat/vB. And so on and so forth.

We can see that at time 21:06 the mempool was nearly empty and all 1 sat/vB transactions got accepted, but this was not the case 30 mins before.

![Screenshot from 2021-07-18 21-55-57.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626625570723/FjlL3UgbZ.png)

Incoming transactions - shows the amount of unconfirmed transactions entering the mempool, measured in vBytes per second (vB/s).

So around 21:32 in this chart, we can see there was a sudden influx of unconfirmed transactions entering the mempool, around ~3,500 vB/s.

If we assume that each transaction is on average ~450 bytes, then the meter above tells us there are ~2 incoming transactions every second.

So now that you are girded with all this data, you can customize your fees accordingly. Once you’ve made the transaction, you can see its status using mempool.space. 😎

So until next time, Set your fees wisely 💸💪
