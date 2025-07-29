---
layout: post
title: "My Summer of Bitcoin Journey: Gearing Up for the Proposal Round"
date: 2024-08-08
author: "Ian Slane"
categories: [Stories, LDK]
---

Hey, bitcoiners and those who are interested in either the Summer of Bitcoin or my learning journey! As I enter the final week of the Summer of Bitcoin Bootcamp, anticipation grows for the upcoming proposal round. Going deeper into the technical side of Bitcoin has been an exciting journey, and yet, my focus now turns squarely to the imminent proposal round tasks. Over the past month, this upcoming session and the days following it have loomed large in my mind, recognizing how pivotal their significance is. Deep down, I know that these moments will mark a turning point, shaping my path toward becoming a bitcoin developer and a better developer in general. While there may be alternative routes to this goal, my determination is steady — I'm committed to the direct path. The acceleration of my learning and advancement as a Rust developer can find no better route than securing an internship. Get excited because beyond this week lies the promise of exciting challenges and limitless growth.

### A Shift in Approach

This week's approach will see a slight shift in organization. While I've consistently engaged with two chapters of Grokking Bitcoin each week, I've encountered a challenge in condensing the multitude of fascinating topics into single articles. Some subjects, such as the intricacies of blocks and their headers, are so deep in complexity that they could each warrant a substantial blog post of their own. For instance, the nuances of block IDs, merkle roots, timestamps, and proof-of-work could easily consume a 30+ minute read. Yet, rather than attempting to cram these expansive topics into one post, I've decided to reserve them for future exploration when time allows. Now, my focus is on actively contributing and participating in open-source projects, a commitment I aim to keep consistently. Therefore, in this week's post, I intend to dig into the project that's the target of my laser eyes, highlighting what intrigues me about it and outlining my starting point.

### Grokking Fedimint

In short, Fedimint offers an innovative solution empowering individuals to establish or join a “community bank” known as a federation, where guardians (key holders) are entrusted with the custody of bitcoin instead of relying on centralized third parties. With seamless integration with the Lightning Network, Fedimint ensures swift, secure, and highly private transactions, elevating the user experience to new heights. As I contemplate potential project proposals for the Summer of Bitcoin, Fedimint stands out as my leading choice, although I'm uncertain of their participation in this year's program. My interest in this project is multifaceted. Firstly, Fedimint has a welcoming and supportive open-source development community, fostering an environment that helps growth and collaboration. Also, their alignment with Rust resonates deeply with my skills and aspirations, further motivating my involvement. Next, Fedimint's focus on scaling Bitcoin aligns perfectly with my initial motivation to delve into coding. Even beyond the Summer of Bitcoin, I'm confident that Fedimint would remain the open-source project I'd chosen to contribute to. Their commitment to maintaining exceptionally high-quality code, characterized by readability and organization, is a testament to their dedication to greatness. By engaging with a team that prioritizes these standards, I'm confident that my coding skills will undergo significant refinement and improvement.

### Documenting the Fedimint-https Repository

Recently, I embarked on a task within the Fedimint-https repository, aiming to draft documentation for its Rust crates. In Rust, a crate serves as a fundamental unit of compilation or packaging, akin to modules or libraries in other programming languages. Specifically, the Fedimint-https repository facilitates a REST API, enabling seamless interaction with the Fedimint-cli across multiple programming languages.

This endeavor serves as an ideal starting point for my journey into the project's development environment and the utilization of the Fedimint-cli. However, venturing into this new area is not without its challenges. As someone relatively new to Nix, configuring it to integrate with the project proved to be a time-consuming endeavor, and a valuable learning experience.

Moreover, navigating through the Fedimint development environment posed its own set of hurdles, primarily due to the absence of comprehensive instructions. Nevertheless, I am undeterred, recognizing these obstacles as opportunities for growth and learning.

My immediate focus now lies in crafting meticulous crate documentation. This involves configuring the .env file correctly, establishing the data directory for the client, and providing illustrative examples of cURL commands and corresponding responses in the README. Adhering to Rust's Rustdoc conventions is imperative to ensure clear and concise function definitions.

Furthermore, I aim to facilitate seamless integration by providing links to all-consuming clients such as Fedimint-ts, Fedimint-go, and Fedimint-py.

While the journey ahead may seem daunting, I'm fueled by the prospect of gaining a deeper understanding of this project's intricacies. This initial start into documenting a small segment of the codebase signifies a significant milestone in my technical journey.

### Looking Ahead

As I end this chapter of my Summer of Bitcoin journey, the road ahead beckons with possibility. In the next blog post, I'll go into what it's like in the proposal round, sharing insights into my selection process and the projects I've chosen to apply for. Narrowing down my options to select projects where I can make a meaningful contribution marks a crucial step in my journey towards becoming a BitDev. Let's get it.
