---
layout: post
title: "Building a P2Pool v2 Simulation: My Work with SoB and NS-3"
date: 2025-07-10
author: Parth Shah
categories: [P2Pool, Stories]
---

Hey! I’m Parth, a developer deeply interested in the **technical foundations of Bitcoin**, especially the inner workings of **Bitcoin Core** and the **Lightning Network**.

Over the past few months, I’ve been focused on understanding Bitcoin at a protocol level — how nodes communicate, how transactions propagate, and how consensus is maintained — all the good stuff under the hood. I’m currently participating in the **Summer of Bitcoin 2025 program**, and I’m super excited to share my journey through it!

## **What is Summer of Bitcoin?**

[Summer of Bitcoin](https://www.summerofbitcoin.org/) (SoB) is a global open-source internship program that helps university students contribute to the Bitcoin ecosystem. It starts with a series of technical assignments covering Bitcoin and Lightning, followed by project proposals and mentorship.

The program is highly selective and is designed to **introduce contributors** to real-world open-source development — especially the kind that powers decentralized systems like Bitcoin.

## What am I working on?

I'm contributing to a project titled **"**[**NS-3 Simulation of P2Pool v2**](https://github.com/p2poolv2/Ns3_simulation)**"** under the [**P2POOL-V2 organization**.](https://github.com/pool2win/p2pool-v2)

The project focuses on **simulating the behavior of P2Pool v2**, a decentralized mining pool protocol, using the **ns-3 network simulator**. The goal is to accurately model key components such as **miner communication, share propagation, DAG-based sharechain construction**, and **block discovery** in a realistic and extensible environment.

The purpose of this simulation is to **identify critical protocol parameters** and gather data to understand **the impact of introducing uncles (or orphaned shares)** on sharechain structure, propagation latency, and overall mining efficiency.

## What's Next?

I’ll continue improving the simulation by refining the sharechain logic, adjusting network parameters, and validating results against expected behavior. Upcoming tasks include:

* Tweaking mining intervals and propagation delays
    
* Adding better logging and visualization of the DAG
    
* Exploring metrics like orphan rate, latency, and fairness
    

I’ll be sharing regular updates as the project progresses — stay tuned for more technical deep dives and insights in upcoming posts!