---
layout: post
title: "My Summer of Bitcoin 2024: A Concluding Reflection on My Journey with Coinswap"
date: 2024-08-28
author: Divyansh Gupta
categories: [Coinswap, Stories]
image: ""
---

As the Summer of Bitcoin 2024 comes to an end, I’ve been reflecting on my journey. It’s been an incredible experience of learning, growth, and contributing to the Bitcoin ecosystem. This program allowed me to dive deep into Bitcoin, work with talented people, and make meaningful contributions.

I’m excited to share the work I’ve done throughout this journey. From the start, I was eager to tackle technical challenges and be part of this community. Looking back, I’m truly grateful for the chance to be involved and contribute to a technology that has a significant impact on the world.

This experience has not only improved my technical skills but also reinforced my belief in the power of open-source collaboration.

## **Brief About Mid-Term Work:**

I explored the BDK codebase, analyzed BDK-based wallets like `Mutiny-Node`, developed essential APIs like `Wallet::init` and `Wallet::load`, and conducted extensive testing to ensure smooth integration and functionality. This work has laid a solid foundation for advancing the Coinswap Wallet project.

For more details, you can refer to my mid-term reflections here: [`My Summer of Bitcoin Journey: Midway Reflections`](https://divyansh123.hashnode.dev/my-summer-of-bitcoin-journey-midway-reflections-1)`.`

# **Post Mid-Term Work**

## **BDK Wallet Infrastructure Postponed**

Following the mid-term evaluations, the focus of our project shifted. The **BDK Wallet Infrastructure**, a crucial but time-consuming component, was **postponed** and pushed to future milestones. This decision was made to concentrate on more immediate issues to meet our current goals, including the public release of version **v0.1.0** and a demonstration of the **Coinswap protocol**. Consequently, my work pivoted from the BDK Wallet project to tackling current challenges and engaging in the review process.

## **My Work Since Then**

With the BDK project on hold, my attention shifted to pressing tasks and ongoing improvements. One of my primary responsibilities has been **developing integration tests** for the **Taker CLI app**. Although the associated PR is still in draft, it has been a valuable experience. As we worked through this, we identified several opportunities for enhancements and refactoring, such as adding new **subcommands** and refining the code structure to make the CLI apps more robust and feature-rich.

## **PR Review Process**

A significant part of my recent work has involved **reviewing PRs**. This process is crucial as it ensures that raw code is polished, maintainable, and integrates well with the existing codebase. PR reviews go beyond just fixing bugs; they ensure that the code is **clear**, **well-documented**, and **future-proof**.

To date, I have reviewed around **five PRs**, which has greatly boosted my confidence in the open-source space. One noteworthy PR, submitted by my mentor, involved **removing the Tokio dependency** to simplify the codebase by making the entire Coinswap processes **synchronous**, as most of its operations need to be done in a sequential way.

`Tokio` is a tool that helps programs run tasks in the background without getting stuck, allowing them to manage multiple tasks at once without slowing down. This change impacted nearly every part of the code, offering me a deep dive into how the Coinswap protocol operates. Although my suggestions were mostly minor, the review provided valuable insights into the protocol.

## **Issues Discovered and Solutions**

During the PR review process, I identified several **internal bugs**:

1. **Extra Address in** `get_next_internal_address` API: This API was returning more addresses than requested. Fixing this issue highlighted the need for extensive **refactoring**, but the fix has been deferred until the **BDK integration**, as BDK will eventually handle this logic more efficiently.
    
2. **Missing Locktime for Transactions:** While reviewing a PR, I discovered that **Contract** and **Funding** transactions lacked a **locktime**, making them vulnerable to **Fee-Sniping Attacks**, where miners replace transactions to gain higher fees. Setting the locktime to the current blockchain height addressed this vulnerability and improved protocol efficiency.
    

## **Community Support:**

One of the most amazing aspects of this journey has been the community support I received from the very beginning, especially as a newcomer to the project and Bitcoin.

My mentors, `Bit-Aloo` and `Raj`, have been exceptional. They encouraged me to come up with my own solutions, which not only improved my decision-making skills but also helped me grow personally. Despite being excellent developers, they always valued my opinions and showed a high level of respect and support. I never felt like just a student intern; they made me feel like a valuable part of the project, which kept me engaged and motivated.

Every Thrusday at 2:30 pm UTC, we held team meetings to discuss our progress and plans for the week. My mentors created an environment where making mistakes and learning from them was part of the process. They always reassured me that no question was too trivial. I feel incredibly fortunate to have had such supportive mentors. I truly believe I wouldn’t have made it through this journey without their guidance.

## **What After Summer of Bitcoin '24**

After my **Summer of Bitcoin** internship, I plan to continue contributing to **Coinswap**. This will involve addressing various issues to speed up the launch of our first version. Following that, I will focus on the **BDK infrastructure** project and review any new PRs that come in.

## **Overall Experience**

As I wrap up this summer, I can honestly say that it has been an incredible journey of learning and growth that I'll always treasure. I'm deeply thankful to the Summer of Bitcoin program for providing me with this unique opportunity.

The regular meetings with my mentors and fellow contributors were invaluable. These sessions, where we discussed our progress and challenges, made me feel supported and part of a team. The guidance I received helped me navigate the complexities of the project and sharpen my skills.

Despite the challenges, this summer has been a rewarding experience, balancing technical growth with real-world application. It has been an adventure that has not only enhanced my understanding of Bitcoin but also strengthened my confidence in tackling complex problems.

To everyone who offered advice, feedback, or even just words of encouragement along the way — **thank you**. Your contributions were instrumental in shaping my understanding and skills, making this journey a true collaboration. I'm grateful for every moment and for the opportunity to be part of this community.
