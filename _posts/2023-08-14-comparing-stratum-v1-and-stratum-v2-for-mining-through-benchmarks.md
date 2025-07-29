---
layout: post
title: "Comparing Stratum V1 and Stratum V2 for Mining (through benchmarks)"
date: 2023-08-14
author: Prisca Maduka
categories: ['Stories', 'Mining']
---

**Introduction:**

Last summer, I had a fantastic opportunity to participate in Bitcoin
development through an internship offered by Summer of Bitcoin (SOB).

Summer of Bitcoin is a global online summer internship program that introduces
university students to Bitcoin open-source development and design. Mentors
guide students in contributing to an open-source Bitcoin project for 12 weeks,
with tracks in Software Development or Design. The program aims to teach about
Bitcoin and open-source development while providing a fulfilling experience,
and participants receive a stipend in BTC. You can visit the Summer of Bitcoin
website to learn more about the program, application process, and the exciting
opportunities available.

During my internship, I worked with the Stratum Development Team. This team of
developers focuses on improving the logic and framework of the Stratum
protocol, ensuring compatibility with both V1 and V2 miners and mining pools.
Working with them provided insights and allowed me to contribute to this
community and learn about Bitcoin.

With support from my mentor Filippo and collaboration from Gabriel, I
conducted a project to compare Stratum V2's superiority to Stratum V1 by
benchmarking their performance.

**Why This Project Matters:**

Stratum V1 has been the standard way for miners to work together for about
8–10 years. It improved how miners cooperate, but it has some limitations.
Stratum V2 aims to address the limitations of v1. It. is a new protocol for
pooled mining that improves the efficiency, security, and decentralization of
bitcoin mining.

Benchmarking the performance of Stratum V2 and V1 is important because it
helps us understand which communication protocol is more effective for mining.
By comparing their performance using real data and key metrics, we can
determine which protocol offers better efficiency, speed, and security. This
information is crucial for miners and developers who want to make informed
decisions about which protocol to use. It ensures that mining operations are
optimized, leading to improved results and a more efficient use of resources.

**Our Goal:**

Through this project, we aim to encourage more individuals to adopt Stratum V2
for mining activities.

**Key Metrics Measured**

We analyzed key metrics to highlight Stratum V2's capabilities. These metrics
include

  * Latency
  * L1 Access (Level 1 Cache)
  * L2 Access (Level 2 Cache)
  * RAM Access
  * Estimated Cycles
  * Instructions per Cycle

Understanding these metrics is crucial to grasping how well Stratum V1 and
Stratum V2 perform. Latency tells us how fast the communication is between
miners and pools. Cache access (L1 and L2) and RAM access affect how quickly
data can be retrieved, and estimated cycles and instructions per cycle give
insights into overall efficiency. By measuring these metrics, we can provide a
comprehensive picture of the strengths and areas of improvement for both
protocols.

**Methodology:**

We took a careful approach to our analysis using two tools that help us
measure and compare performance: criterion and iai. These tools were chosen
for their ability to ensure meticulous and consistent benchmarking,
contributing to the accuracy and reliability of our findings.

  * criterion: We used criterion specifically to measure latency, a key metric in our analysis. It is a widely used benchmarking library for the Rust programming language. It collects and stores statistical information from run to run, automatically detects performance regressions, and measures the performance of code.
  * iai: For the remaining metrics, namely L1 Access, L2 Access, RAM Access, Estimated Cycles, and Instructions per Cycle, we turned to iai. This tool excels at gauging the the system requirements of functions that are being benchmarked.

**Our Findings**

  * **Latency:** The SV2 connection process showed a significant improvement in latency compared to the SV1 connection process. There was. approximately 58.36% improvement.
  * **Instructions:** Stratum V2 did much better in processing instructions, being 53.12% more efficient than Stratum V1.
  * **L1 Accesses:** Stratum V2 was also better at getting data from its first-level memory, with a 49.81% improvement.
  * **L2 Accesses:** Both SV1 and SV2 had similar L2 cache accesses, with sv2 being slightly higher by 20.00%.
  * **RAM Accesses:** Stratum V2 was really good at using its main memory, using 61.23% less than before, which made things more efficient.
  * **Estimated Cycles:** Stratum V2 was found. to be 44.51% faster in terms of the number of cycles it took, meaning it ran more smoothly and quickly.

You can run this benchmark and the results for yourself by visiting this page
put together by Gabriel.

**What This Means**

Beyond the numbers, our findings have important implications for the world of
cryptocurrencies:

**Boosted Efficiency:** Stratum V2's strong performance suggests it can speed
up and make cryptocurrency transactions more efficient.

**Smart Resource Usage:** The reduced need for RAM accesses and improved
estimated cycles hint at potential energy and resource savings. This is
especially important for eco-friendly blockchain environments.

**Enhanced User Experience:** The improved processing efficiency and reduced
access times demonstrated by Stratum V2 could lead to a smoother and faster
experience for cryptocurrency users. Transactions and mining activities could
occur more swiftly, enhancing user satisfaction.

**Scalability Potential:** The gains in efficiency showcased by Stratum V2's
performance metrics could indicate its potential to handle larger volumes of
transactions and mining activities. This scalability could be vital as the
cryptocurrency ecosystem continues to grow.

**Conclusion:**

Our results strongly support using Stratum V2 for mining, making it faster,
safer, and more spread out. Our project’s results are like a helpful guide for
miners, developers, and others involved. It helps them understand why using
Stratum V2 is a smart move, pushing forward new and exciting changes in how we
mine Bitcoin.

My experience was nothing short of exciting. And yes, I had some challenges,
but I had the support of my mentor, Gabriel and the whole team. I learned so
much about bitcoin, especially bitcoin mining process, I learned teamwork and
collaboration, and also rust programming language. I am so grateful for the
opportunity.

**References :**

Braiins. (n.d.). Stratum V1 Documentation. https://braiins.com/stratum-v1/docs

Braiins. (n.d.). Stratum V2. https://braiins.com/stratum-v2

Bitcoin Magazine. (n.d.). Stratum V2 Will Save Bitcoin Mining.
https://bitcoinmagazine.com/business/stratum-v2-will-save-bitcoin-mining

Swan Bitcoin. (n.d.). The Bullish Case for Stratum.
https://www.swanbitcoin.com/the-bullish-case-for-stratum/

Summer of Bitcoin. (n.d.). About. https://www.summerofbitcoin.org/about

Bheisler, B. (n.d.). Criterion.rs Documentation.
https://bheisler.github.io/criterion.rs/book/index.html
