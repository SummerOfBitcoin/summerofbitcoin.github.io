---
layout: post
title: "Summer of Bitcoin: Building Scalable Infrastructure for Bitcoin Education and Knowledge Extraction"
date: 2026-08-17
author: Rasesh Shetty
categories: [Stories]
---

# My Summer of Bitcoin Journey: Architecting for Scale and Education

When I began my Summer of Bitcoin journey, the goal was ambitious: to build out robust, fault-tolerant infrastructure that could power not just a transcription engine, but an entire educational ecosystem for Bitcoin. Over the past few months, this project evolved into three major pillars: a highly modular **Transcription Engine**, an interactive coding platform called ***Hello Bitcoin***, and an intelligent **Data Scraper**. 

Here is the story of what I set out to do, the technical hurdles I overcame, and the final systems we shipped.

---

## 1. The Transcription Engine

The initial state of the transcription service needed a major overhaul. My first priority during the mid-term was to establish a **fault-tolerant, server-resumable pipeline**. Transcribing large audio files is resource-intensive, and a single failure shouldn't mean starting from scratch. I built a robust pipeline that flawlessly executes all stages for a single transcript while ensuring that if something breaks, the system knows exactly where to pick back up.

### Architectural Modularity & Dynamic Integrations
Instead of hardcoding models, I completely generalized the ASR (Automatic Speech Recognition) integration. By moving to a modular service provider architecture, adding new models became a breeze:
* **Dynamic LLM Integration:** The engine now reads the type of LLM dynamically from environment variables, removing hardcoded dependencies.
* **CLI & Drop Features:** I added CLI choices for selecting ASR providers on the fly and modified the initial setup to seamlessly "drop in" new service providers. 
* **Expanding the AI Arsenal:** By the end of the term, I successfully integrated **Ollama** and **Vibevoice**, proving the effectiveness of the modular design.

### Translation & Global Scaling
Bitcoin is a global movement, so transcription alone wasn't enough. I introduced a comprehensive **Translation Feature** utilizing Sarvam AI, with Gemma models serving as a reliable fallback. I built out new API endpoints dedicated to these translation capabilities, ensuring the engine could serve a diverse, international user base.

### Strengthening the Core Pipeline
A significant portion of my time went into making the engine enterprise-ready:
* **Structured Exception Handling:** I introduced a hierarchical exception module. Now, when the ASR or Translation engines fail, the system categorizes the error intelligently rather than just throwing generic faults.
* **Concurrency & State Management:** I resolved critical bugs where transcription statuses would hang on "pending" despite failing. Through rigorous code reviews (including Copilot reviews), I implemented strict **global locking** and ensured thread safety across the board.
* **Hardware Agnosticism:** For my final challenge, I proved the engine's versatility by successfully deploying and running the transcription workflows on both an **AWS EC2 instance** and a **Raspberry Pi**.

---

## 2. Hello Bitcoin: Building an Educational Platform for Bitcoiners

While the transcription engine processed knowledge, *Hello Bitcoin* was designed to teach it. I built this interactive education platform from a simple MVP into a fully featured, robust system.

### The Foundation
I started with the core infrastructure: setting up basic authentication and seeding an initial problem list. Then I engineered the "Judge Core", a **Docker Sandbox Runner** that safely executes user-submitted code in a completely isolated environment. Building on that foundation, I integrated async workers to handle code submission processing and implemented **Server-Sent Events (SSE)** so users could watch their test results stream in live.

### Security, Admin, and Scale
During the second half of the summer, I focused heavily on hardening the platform and preparing it for real-world traffic:

* **Enhanced Security:** Upgraded the authentication flow by implementing Refresh Tokens and Redis-backed JTI (JSON Web Token ID) revocation to effectively prevent token hijacking.
* **Comprehensive Admin Tools:** Built a full Admin Panel featuring a Polygon-style editor for Problem CRUD operations, alongside sophisticated User Management for handling roles and promotions.
* **Production Readiness:** Wrapped the entire architecture in a robust Docker Compose stack. I also wrote a comprehensive test suite with high Pytest coverage and conducted extensive stress testing to ensure the judge wouldn't crash under load.
* **Curriculum Alignment:** Finally, I modified the underlying database schema and UI to directly map coding challenges to specific Bitcoin conferences and book chapters, creating a deeply contextual and relevant learning experience for users.

---

## 3. The Knowledge Scraper Module

To feed our educational platforms with the latest data, I built a dedicated web scraper. Initially starting as a basic extraction feature, I completely refactored it for the final term. 

I implemented a clean, **Object-Oriented architecture** featuring a base Parent Class, with specialized child classes dedicated specifically to parsing GitHub repositories and extracting from general websites (online books, blogs, and major BTC sites). This structure ensures that as our data sources grow, adding a new scraper requires minimal boilerplate.

---

## Major Technical & Design Decisions

1. **Dockerized Sandboxing for Code Execution:** Running untrusted user code is incredibly dangerous. Choosing Docker for the *Hello Bitcoin* judge allowed me to enforce strict resource limits (memory/CPU) and network isolation.
2. **Modular Provider Pattern:** By treating AI models (ASR, LLMs, Translations) as interchangeable plugins rather than core dependencies, the Transcription Engine can pivot to newer, cheaper, or faster models instantly without refactoring the core pipeline.
3. **Redis for State & Auth:** Utilizing Redis wasn't just for queuing async code submissions; it became crucial for security (JTI revocation) and fast state retrieval, keeping the app stateless and horizontally scalable.
4. **Hierarchical Exceptions:** Creating a custom exception tree allowed the outer layers of the application to gracefully handle errors (e.g., retrying an API limit error vs. failing fast on a missing file) without leaking internal logic.

---

## PRs & Deliverables

Below are the key artifacts, pull requests, and commit milestones generated during this program:

### Transcription Engine
* **Core Architecture & Fault Tolerance:** [PR #15 — Fault-tolerant and resumable transcription pipeline](https://github.com/genesis-kb/transcription_engine/pull/15)
* **Dynamic LLM & Modular ASR Integration:** [PR #28 — Modular service integration & dynamic provider choice](https://github.com/genesis-kb/transcription_engine/pull/28)
* **Translation Feature (Sarvam/Gemma):** [PR #34 — Translation pipeline & API endpoints](https://github.com/genesis-kb/transcription_engine/pull/34)
* **Test Suit, Thread Safety & Global Locking Fixes:** [PR #38 — State safety & test suite](https://github.com/genesis-kb/transcription_engine/pull/38)
* **Hierarchical Exception Handling:** [PR #35 — Structured exceptions module](https://github.com/genesis-kb/transcription_engine/pull/35)
* **Ollama and Vibevoice Integration:** [PR #66 — Vibevoice service & Ollama Gemma LLM support](https://github.com/genesis-kb/transcription_engine/pull/66)

### Hello Bitcoin Platform
* **MVP & Core Infrastructure:** [Core Infra, Auth, Problem List & Seeding](https://github.com/genesis-kb/hello-bitcoin/commit/61188413bfbca920887f6935dfdf5b9fd3b920c3)
* **Docker Sandbox Runner:** [Isolated Code Execution Engine (The Judge Core)](https://github.com/genesis-kb/hello-bitcoin/commit/c8f5e6174968cba39f253d0d7441e564262b17f2)
* **Code Submission Processing:** [Asynchronous Queue & Worker Integration](https://github.com/genesis-kb/hello-bitcoin/commit/7660b7612d782634a5179fd6b90970d29fa9e10f)
* **Live Result Streaming:** [Real-time Test Updates via Server-Sent Events (SSE)](https://github.com/genesis-kb/hello-bitcoin/commit/d59b2a639ca60fb609803e9753bd9a8c0d6959fd)
* **Authentication & Token Security:** [Refresh Tokens & Redis-backed JTI Revocation](https://github.com/genesis-kb/hello-bitcoin/commit/d0127a80418b0c19bf72c497bea6b1330dccbc4f)
* **Admin Polygon Editor:** [Problem CRUD & Polygon-style Testcase Editor](https://github.com/genesis-kb/hello-bitcoin/commit/c0bb02c2c29e82ba19e73a5777f5a90f78357d0f)
* **Admin User Management:** [Role-Based Access Control & User Promotion](https://github.com/genesis-kb/hello-bitcoin/commit/b4d0356038bf5c9fb108be4d364c479486aff7ce)
* **Containerization & Testing:** [Docker Compose Stack & Comprehensive Test Suite](https://github.com/genesis-kb/hello-bitcoin/commit/ed701c57b1c7e9eb36166071a08983af0806198b)
* **Stress Testing & Bug Fixes:** [Load Testing, High Pytest Coverage & Resilience](https://github.com/genesis-kb/hello-bitcoin/commit/3a81d0a79b4f6a49db5018cd410ed65dabd7b663)
* **Curriculum Alignment:** [Conference & Book Chapter Schema Mapping & UI Updates](https://github.com/genesis-kb/hello-bitcoin/commit/0163c07f0131de452c462b75869e934a4ccdd9c0)

### Scraper Module
* **Scraper Architecture (GitHub & Web):** [GitHub & Web Scraper Implementation](https://github.com/genesis-kb/scrapers/commit/52d03695208f12bfb76c5e67ea8a0a1357da2eb5)

---

## What I Learned & What's Next

This summer was a masterclass in building systems that last. It's one thing to write a script that transcribes a video; it's entirely another to build a thread-safe, modular engine that can resume upon failure, run on a Raspberry Pi, and dynamically swap AI models. Building the Docker Sandbox taught me the intricacies of system security and asynchronous task management.

Looking ahead, I plan to continue maintaining the Hello Bitcoin platform, expanding the problem sets, and refining the Docker Judge to support even more programming languages. 

I am incredibly grateful to my mentors and the Summer of Bitcoin community for their guidance, code reviews, and support throughout this incredible journey!
