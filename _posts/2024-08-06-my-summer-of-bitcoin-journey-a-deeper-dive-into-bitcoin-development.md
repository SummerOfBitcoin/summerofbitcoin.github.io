---
layout: post
title: "My Summer of Bitcoin Journey: A Deeper Dive into Bitcoin Development"
date: 2024-08-06
author: "Ian Slane"
categories: [Stories, LDK]
---

Since I published my first article last week, the pace of my learning journey has caught a ton of momentum, evoking both excitement and a touch of apprehension. While I've managed to grasp the basics of bitcoin, it's become increasingly apparent that the depth of this technology is as vast as it is fascinating. With every revelation and challenge encountered, it feels as though I've only skimmed the surface. Working through avenues ranging from cryptography coding projects to honing my skills in technical writing and preparing for project proposals has proven to be tough. Yet, among the mountain of tasks and responsibilities, I find a deep sense of confidence and passion driving me forward. Balancing these objectives alongside the final semester of my college education adds another layer of complexity to my journey. Yet, it's these challenges that fuel my determination and deepen my commitment to becoming a BitDev.

In this article, I'll talk about my initial experiences during the first week of Summer of Bitcoin and share insights gained so far. Then, I'll explore the intricacies of address generation, shedding light on the process and its significance. Next, we'll dive into the practice of enhancing security by incorporating a checksum into the public key hash before encoding, clarifying how this measure fortifies address validation and ensures accuracy during transactions. Additionally, I'll talk about a challenge I'm facing in finding a suitable project, providing insight into the ongoing endeavors within my journey of bitcoin development.

### Summer of Bitcoin Week 2

Continuing to the second week of the Summer of Bitcoin boot camp, I felt that each interaction and discussion took me further into understanding this week's chapters. This was the first session where we interacted with other applicants, and it provided an opportunity to put theory into practice and engage with my peers. In our first meeting since intro day, we dove headfirst into digital signatures and the transformation from private key to bitcoin address. I even felt more confident in my understanding of the material because I wrote about the concepts in my previous article. I often found myself aiding and clarifying concepts for my peers, which felt pretty good. After the meeting, a group session was held where a bitcoin core maintainer assisted in addressing questions. Also, we had the opportunity to hear from a former intern. It was super great to hear from him, as he has since secured a position at a bitcoin-only company. Listening to someone who shared similar experiences was super inspiring, igniting my enthusiasm for the upcoming proposal round, where I'm ready to compete for a spot!

### Bitcoin Address Generation In-Depth

Address generation relies on a process known as Base58check encoding, which plays a vital role in maintaining security, enhancing usability, and safeguarding user privacy within bitcoin. This encoding procedure goes through four distinct stages, each crafted to ensure the integrity of the addresses.

The first step in Base58check encoding involves appending a version number to the beginning of the public key hash (PKH). Typically represented by a single 0 byte, this version number serves as a marker to differentiate between various address types or network protocols employed within transactions.

Following the addition of the version number, the versioned PKH undergoes a double hashing process utilizing the SHA-256 cryptographic function. This entails subjecting the versioned PKH to two consecutive rounds of hashing: SHA256(SHA256(versioned PKH)). By subjecting the PKH to this double-hashing procedure, the encoding process not only strengthens the security of the resulting address but also guarantees the uniqueness of the subsequent checksum.

The third phase of Base58check encoding entails computing a checksum from the second hash obtained through the double-hashing process. This checksum, including the first four bytes of the second hash, plays a key role akin to padding in cryptography. It ensures that the resulting address adheres to specific formatting requirements and facilitates robust error detection mechanisms.

Finally, the versioned PKH, along with the checksum, is encoded using Base58, a character set that excludes ambiguous characters like '0' and 'O', 'I' and 'l', thus enhancing readability and usability. Before encoding, the leading version number is removed to simplify the address format.

In summary, Base58check encoding serves as the backbone of bitcoin address generation, providing a secure and reliable method for creating addresses. These addresses are resilient against tampering and errors. Through its careful design and implementation, Base58check encoding reinforces the integrity of the network, ensuring that users can transact with confidence and privacy.

### Ensuring Address Accuracy with Checksums

Upon receiving a transaction, the recipient must obtain the sender's Public Key Hash (PKH). Fortunately, the Base58check decoding process allows this by enabling the reversal of the encoding process. This not only retrieves the PKH from the decoded Base58 address but also verifies the integrity of the input data, guarding against potential errors.

The decoding and verification process starts with the sender, who receives the address intended for the transaction. The first step entails decoding the Base58 encoding, revealing a versioned PKH. This versioned PKH then undergoes the same SHA256 double hashing process applied during encoding. If the resulting checksum matches the original checksum, it signifies that the address is valid and free from any input errors. Conversely, a mismatch between the checksums indicates potential corruption or typing errors in the address. In this case, the sender's wallet is warned of the issue, preventing the transmission of bitcoin to a compromised address.

By leveraging the checksum verification mechanism within the Base58check decoding process, users can confidently transact knowing that their addresses are accurately interpreted and validated, thereby ensuring the security and integrity of the network! Pretty cool huh?

### Navigating Ambitions

When I started documenting my journey, I pictured a section dedicated to showcasing my coding prowess through snippets of my work. Last week, I started writing an encoding/decoding program in Rust, a task that proceeded smoothly. Encouraged by this success, I set my sights on a more ambitious project: developing a program capable of generating private and public keys, ultimately culminating in the derivation of bitcoin addresses through Base58check encoding. However, the intricacies of generating a public key using elliptic curve cryptography proved to be a tough challenge. While the idea of building fundamental components of bitcoin from scratch held a certain appeal, I soon realized that the investment of days and hours into this endeavor lacked practical utility. After consideration, I ended up redirecting my efforts toward finding a project aligned with my career aspirations that would yield more dividends. While I had hoped to include a code snippet to illustrate my journey, I also wanted to candidly discuss the challenges encountered in identifying a suitable project and effectively managing my time. In the coming week, I'll introduce code excerpts and elaborate on the project I've chosen to undertake, clarifying the motivation behind my decision.

### Embracing the Journey

Overall, my journey into bitcoin development has been an exciting blend of learning, challenge, and growth. As I go deeper into the technical side of bitcoin, I'm continuously amazed by its complexity and potential. From navigating the Summer of Bitcoin to exploring the nuances of address generation and checksums, each step has shone new facets of this fascinating field. Despite the hurdles and uncertainties along the way, my passion for becoming a BitDev only grows stronger. As I prepare to tackle the next phase of my journey, I am eager to confront new challenges, refine my skills, and ultimately make meaningful contributions to the bitcoin ecosystem.
