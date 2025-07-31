---
layout: post
title: "A Beginner's Guide to Contributing to LDK Node"
date: 2024-08-16
author: "Ian Slane"
categories: [Tutorials, LDK]
---

This article provides a guide for new contributors interested in contributing to the Lightning Dev Kit (LDK) and LDK Node.

### Getting Started with LDK

**What is LDK?**

LDK (Lightning Dev Kit), also known as rust-lightning on GitHub, is a high-performance and versatile implementation of the Lightning Network protocol, written in Rust. It simplifies the process of integrating Lightning Network functionalities into various projects, offering tools for managing payments, channels, and network interactions. LDK Node is a self-custodial Lightning node library built with LDK and BDK. It's designed to offer a compact, user-friendly interface that simplifies setting up and operating a Lightning node, complete with an integrated on-chain wallet. While it prioritizes simplicity, LDK Node is also modular and adaptable, making it versatile enough to suit various needs and applications.

**Setting up your Development Environment**

Before you begin, you'll need the contributing guide, Rust, and Git. The contributing guide outlines essential information on project structure, coding standards, formatting, and best practices.

**Cloning the Repository**

First, log into your GitHub account and fork the LDK Node repo. Then, in your terminal, clone your fork using the following command, replacing `your-user-name` with your GitHub username:

```
git clone git@github.com:your-user-name/ldk-node.git
cd ldk-node
```

**Building the Project**

Before working on an issue, it's a good idea to build the project and run the tests to verify your setup. To do that, run the following commands in your terminal:

```
cargo build
cargo test
```

### Making Your First Contributions

**Finding a Good First Issue**

Visit the Issues page on GitHub and look for issues labeled "good first issue" to find tasks suitable for newcomers. You can start with smaller tasks or even documentation updates to understand the LDK Node codebase and get familiar with the project's structure. If you have more in-depth questions about LDK or LDK Node, many discussions about LDK take place on the LDK Discord in the #ldk-dev channel.

**Create a Branch**

Create a branch specific to the issue you're working on. A good practice is to state the year, month, and issue in your branch name. For example: `2024–07-introduce-SendingParameters`.

```
git checkout -b 2024-08-my-first-contribution
```

Make your changes and commit them.

```
git add -p file.rs
git commit
```

Before writing your commit message, it's a good idea to look up what makes a good commit message.

**Time to push your branch!**

```
git push origin 2024-08-my-first-contribution
```

**Creating a Pull Request**

Navigate to your GitHub page where your fork is located and create a pull request. Provide a detailed title and description of your changes and reference the issue you're addressing. Finally, submit the pull request and be open to feedback from reviewers.

### Join the Discussion

Participating in discussions on GitHub is invaluable. Engaging with other contributors can provide insights and help you learn more about the project. If you come across any challenges, don't hesitate to reach out on Discord and ask for help.

### Keep Contributing

As you become more comfortable with LDK and LDK Node, consider working on more complex issues or features. Your contributions can have a significant impact on the project's development.
