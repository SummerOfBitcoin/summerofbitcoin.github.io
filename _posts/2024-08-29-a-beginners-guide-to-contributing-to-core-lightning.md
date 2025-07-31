---
layout: post
title: "A Beginner's Guide to Contributing to Core Lightning"
date: 2024-08-29
author: "Max Rantil"
categories: [Tutorials, Core Lightning]
---

## INTRODUCTION

Core Lightning is an essential project in the Bitcoin ecosystem, providing a robust implementation of the Lightning Network protocol. This article aims to guide newcomers, particularly open-source enthusiasts, through the process of contributing to Core Lightning.

This guide will walk you through the necessary steps to start contributing, from understanding the basics of Core Lightning to submitting your first pull request.

## 2. UNDERSTANDING CORE LIGHTNING (CLN)

**What is CLN?** Core Lightning, formerly known as c-lightning, is an implementation of the Lightning Network protocol. CLN is developed by Blockstream and is written primarily in C, with some components in Python. On top of CLN you can easily write plugins and those can be written in almost any language.

**Its role in the Bitcoin ecosystem:** The Lightning Network addresses Bitcoin’s scalability challenges by enabling off-chain transactions. Core Lightning plays a crucial role in this ecosystem by providing:

*   A full implementation of the Lightning Network specification
*   A platform for running Lightning nodes
*   Tools for developers to build Lightning-enabled applications

**Key features and benefits:**

*   **Speed:** Enables near-instantaneous Bitcoin transactions
*   **Scalability:** Allows for a high volume of transactions without congesting the main Bitcoin blockchain
*   **Low fees:** Significantly reduces transaction costs compared to on-chain transactions
*   **Flexibility:** Provides a robust API for developers to build upon
*   **Privacy:** Enhances transaction privacy through its off-chain nature

Core Lightning is known for its focus on security, reliability, and adherence to the Lightning Network standards. It’s a popular choice for developers and businesses looking to integrate Lightning functionality into their Bitcoin-related projects.

By contributing to Core Lightning, you’ll be working on technology that’s at the forefront of Bitcoin’s evolution, helping to make transactions faster, cheaper, and more accessible to people around the world.

## 3. PREREQUISITES

Before you start contributing to Core Lightning, it’s important to ensure you have the necessary skills and tools. This section will cover the technical requirements and the development environment setup.

**Required technical skills:**

*   A solid understanding of C programming is recommended, as CLN is primarily written in C. However, you can develop plugins for CLN in various other languages as well.
*   Familiarity with Bitcoin concepts: Understanding of Bitcoin transactions, blockchain, and basic cryptography is helpful.
*   Knowledge of networking protocols: Familiarity with TCP/IP and peer-to-peer networks is beneficial.
*   Git version control: Proficiency in using Git for source code management is crucial.
*   Linux/Unix command line: Comfort with command-line interfaces is important, as much of the development and testing occurs in terminal environments.

## 4. GETTING STARTED

Now that you have the prerequisites in place, let’s go through the process of setting up Core Lightning for development.

**Forking the Core Lightning repository:**

1.  Go to the Core Lightning GitHub repository: `https://github.com/ElementsProject/lightning`
2.  Click the “Fork” button in the top-right corner to create your own copy of the repository.
3.  Clone your forked repository to your local machine:
    ```
    git clone https://github.com/YOUR_USERNAME/lightning.git
    cd lightning
    ```

**Setting up the local development environment:**

1.  Add the original repository as a remote to keep your fork up to date:
    ```
    git remote add upstream https://github.com/ElementsProject/lightning.git
    ```
2.  Install additional Python dependencies (docs: `https://docs.corelightning.org/docs/installation`) :
    ```
    poetry shell
    poetry install
    ```

**Building and running Core Lightning:**

Configure the build:
```
./configure
```
Build Core Lightning:
```
make -j$(nproc)
```
Run the unit tests to ensure everything is working correctly:
```
make check
```
Running Core Lightning development environment docs: `https://docs.corelightning.org/docs/developers-guide`

Congratulations! You now have a working development environment for Core Lightning. This setup allows you to make changes to the code, test your modifications, and prepare contributions to the project.

Remember to keep your fork updated with the upstream repository:
```
git fetch upstream
git merge upstream/master
```
In the next section, we’ll dive into understanding the codebase structure and key components of Core Lightning.

## 5. UNDERSTANDING THE CODEBASE

To effectively contribute to Core Lightning, it’s crucial to understand its structure and components. This section will provide an overview of the project’s organization and coding standards.

**Overview of the project structure:** The Core Lightning repository is organized into several key directories:

*   `lightningd/`: Contains the main daemon code
*   `common/`: Shared code used across different parts of the project
*   `wire/`: Defines the wire protocol for communication between Lightning nodes
*   `bitcoin/`: Bitcoin-specific code and utilities
*   `ccan/`: The C Code Archive Network, a collection of reusable C modules
*   `plugins/`: Contains various plugins that extend Core Lightning functionality
*   `cli/`: Command-line interface tools
*   `doc/`: Documentation files
*   `tests/`: Unit and integration tests

**Key components and their functions:**

1.  **Lightningd:** The main Lightning Network node implementation
2.  **Gossipd:** Handles the gossip protocol for sharing channel information
3.  **Channeld:** Manages individual payment channels
4.  **HSM (Hardware Security Module):** Deals with key management and signing
5.  **Onchaind:** Monitors and manages on-chain transactions
6.  **Openingd:** Handles the channel opening process
7.  **Closingd:** Manages channel closing operations

**Coding standards and best practices:**

*   **CODE STYLE:** Core Lightning follows a specific C coding style. Familiarize yourself with it: `https://docs.corelightning.org/docs/coding-style-guidelines`
*   **DOCUMENTATION:** Comment your code, especially for complex logic. Update relevant documentation when making changes.
*   **TESTING:** Write pytests (blackbox tests) or unit tests for new functionality. Study earlier pytests and the `setup_regtest.sh` script to understand more on how to write tests.
*   **COMMIT MESSAGES:** Write clear, concise commit messages. Start with a short (50 chars or less) summary line. Follow with a blank line and then a more detailed explanation if necessary.
*   **ERROR HANDLING:** Use appropriate error codes and messages. Ensure proper cleanup in error cases.
*   **MEMORY MANAGEMENT:** Be mindful of memory allocation and deallocation. Much of the allocation is done with `tal` — tree allocation. Study that for a bit first if you are unfamiliar with it. Use tools like Valgrind to check for memory leaks. To run a test you can use the command: `VALGRIND=1 poetry run pytest -s 'tests/test_connection.py::test_alt_addr_connection_scenarios[standard]' -v --log-cli-level=INFO`
*   **PORTABILITY:** Write code that is portable across different Unix-like systems.
*   **SECURITY:** Always consider security implications of your changes.

Understanding these components and following the project’s coding standards will help you make valuable contributions that are more likely to be accepted by the Core Lightning maintainers.

## 6. FINDING ISSUES TO WORK ON

As a new contributor, it’s important to start with manageable tasks that align with your skills and the project’s needs. Here’s how to find suitable issues:

**Navigating the issue tracker:**

1.  Go to the Core Lightning GitHub repository’s “Issues” tab.
2.  Use the search bar to filter issues based on your interests or skills.
3.  Look for issues marked as “good first issue” or “help wanted”.

**Understanding labels and priorities:** Core Lightning uses various labels to categorize issues. Some important ones include:

*   “good first issue”: Suitable for newcomers to the project
*   “help wanted”: Actively seeking community contributions
*   “bug”: Indicates a problem that needs fixing
*   “enhancement”: Suggests an improvement to existing functionality
*   “documentation”: Related to improving or updating documentation

**Choosing beginner-friendly issues:**

1.  Start with issues labeled “good first issue” or simple bug fixes.
2.  Read through the issue description carefully to understand the requirements.
3.  Check if the issue is already assigned to someone or if there’s an ongoing discussion.
4.  If you’re unsure about an issue, ask for clarification in the comments before starting work.

**Tips for selecting an issue:**

1.  Look if there are any bounties open for a fix at: `https://community.sphinx.chat/workspace/bounties/cmg6oqitu2rnslkcjbqg`
2.  Choose issues that match your skill level and interests.
3.  Look for issues that have been recently updated or have active discussions.
4.  Avoid issues that have been stale for a long time without any activity.
5.  If you’re unsure about your ability to complete an issue, it’s okay to ask for guidance.

Once you’ve found an issue to work on, follow these steps to make your contribution:

**Creating a new branch:**

1.  Ensure your local master branch is up to date:
    ```
    git checkout master
    git pull upstream master
    ```
2.  Create a new branch for your work:
    ```
    git checkout -b your-branch-name
    ```
3.  Use a descriptive name for your branch, e.g., “fix-memory-leak-issue-123”.

**Making changes and testing:**

1.  Make the necessary code changes to address the issue.
2.  Write or update tests as needed.
3.  Run the test suite to ensure your changes don’t break existing functionality: `make check`
4.  Make sure your code follows the project’s coding standards.
5.  Commit your changes with a clear, descriptive commit message.

**Submitting a pull request:**

1.  Push your branch to your forked repository: `git push origin your-branch-name`
2.  Go to the Core Lightning GitHub repository and click “New pull request”.
3.  Select your branch and provide a clear description of your changes.
4.  Reference the issue number in your pull request description (e.g., “Fixes #123”).
5.  Submit the pull request for review.

**After submitting:**

*   Be responsive to any feedback or questions from reviewers.
*   Make requested changes promptly if any are suggested.
*   Be patient, as the review process can take time.

## 7. COMMUNICATION AND COMMUNITY

Effective communication is crucial when contributing to open-source projects. Here’s how you can engage with the community:

Joining relevant communication channels at `https://github.com/ElementsProject/lightning?tab=readme-ov-file#project-status`

**Etiquette for asking questions and seeking help:**

1.  **Search first:** Before asking a question, search existing resources (documentation, issues, mailing list archives) to see if it has been answered before.
2.  **Be specific:** Clearly describe your problem or question, providing relevant details and context.
3.  **Show effort:** Demonstrate what you’ve already tried or researched.
4.  **Be patient:** Remember that contributors are often volunteers and may not respond immediately.
5.  **Be respectful:** Always maintain a polite and professional tone in your communications.

**Participating in discussions and code reviews:**

1.  **Share your insights:** Don’t hesitate to contribute to discussions, even if you’re new.
2.  **Be constructive:** When providing feedback, focus on the code or idea, not the person.
3.  **Ask questions:** If you don’t understand something in a pull request or discussion, ask for clarification.
4.  **Learn from others:** Pay attention to code reviews on other pull requests to learn best practices.

**Continuous Learning and Growth**

*   **RESOURCES FOR DEEPER UNDERSTANDING OF LIGHTNING NETWORK:**
    1.  Read the Lightning Network white paper: `https://lightning.network/lightning-network-paper.pdf`
    2.  Study the BOLT specifications: `https://github.com/lightning/bolts`
    3.  Explore resources on `https://delvingbitcoin.org/` and `https://bitcoinops.org/`
*   **STAYING UPDATED WITH PROJECT DEVELOPMENTS:**
    1.  Regularly check the Core Lightning GitHub repository.
    2.  Follow key contributors on social media like Nostr and Twitter.
    3.  Attend Bitcoin and Lightning Network conferences or meetups.
*   **ADVANCING TO MORE COMPLEX CONTRIBUTIONS:**
    1.  Start with small tasks and gradually take on more complex issues.
    2.  Study the codebase in-depth.
    3.  Propose new features or improvements.
    4.  Offer to mentor newer contributors as you gain experience.

## Conclusion

CONTRIBUTING TO CORE LIGHTNING IS AN EXCITING OPPORTUNITY TO IMPACT THE FUTURE OF BITCOIN AND THE LIGHTNING NETWORK. HERE’S A RECAP OF THE CONTRIBUTION PROCESS:

1.  Set up your development environment
2.  Understand the codebase structure and standards
3.  Find suitable issues to work on
4.  Make your changes and submit pull requests
5.  Engage with the community and participate in discussions
6.  Continuously learn and grow your skills

Remember that every contribution, no matter how small, is valuable. The Core Lightning community welcomes your contributions and is here to support your journey as an open-source contributor.
