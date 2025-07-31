---
layout: post
title: "Building AI-Powered Bitcoin Development Tools: My Mid-Term Journey in Summer of Bitcoin 2025"
date: 2025-07-10
author: Shashank Shekhar Singh
categories: [AI, ChatBTC, Stories]
image: ../assets/images/blog_content/2025-07-10-building-ai-powered-bitcoin-development-tools-my-mid-term-journey-in-summer-of-bitcoin-2025_116fa7a6.png
---


Share

Hi! I’m Shashank Shekhar Singh, a Mechanical Engineering student at IIT BHU,
currently in my Junior year. I am passionate about the intersection of
artificial intelligence and Bitcoin development. Having contributed to the
Bitcoin ecosystem in Summer of Bitcoin 2024 with libbitcoin, I returned this
year with an even more ambitious vision: building AI-powered tools that could
revolutionize how developers interact with Bitcoin codebases and knowledge.

As I reach the halfway point of my Summer of Bitcoin 2025 journey, I’m excited
to share the progress on my project: developing a comprehensive AI-powered
tool ecosystem specifically designed for Bitcoin development. When I submitted
my proposal for “AI-Powered Development Assistant”, I envisioned creating a
Retrieval-Augmented Generation (RAG) system that would transform how
developers access Bitcoin-specific knowledge, documentation, and community
discussions.

# The Original Vision: Democratizing Bitcoin Development

My project proposal was born from a simple observation: **Large Language
Models often struggle with Bitcoin-specific APIs and projects, frequently
hallucinating function arguments and API parameters.** I set out to build an
AI system that would have genuine Bitcoin-specific knowledge — understanding
interfaces, APIs, coding standards, and development discussions at a deep
level.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*iRb8cC3bFqHfj7WmCHsJfg.png"/>
</figure>

Original Architecture

# Project Goals: From Vision to Reality

My original proposal outlined five interconnected goals, each building toward
a comprehensive Bitcoin development assistant. Let me walk you through what I
planned and what I’ve achieved so far.

# Goal 1: Bitcoin Project-Specific Code and Documentation Ingestion

**Original Plan** : Create a RAG system that can ingest and index any Bitcoin
project, understanding coding styles, dependencies, and documentation through
AST parsing.

**What I’ve Built** : I’ve successfully implemented both approaches I
originally proposed, with some improvements to be made:

  * **Graph-based approach** : Developed comprehensive systems using both Neo4j and/or ArangoDB that parse code repositories into structured knowledge graphs, extracting functions, classes, dependencies, and relationships.
  * **Vector embeddings approach** : Built FAISS-based systems that provide rich semantic understanding of Bitcoin Mailing Lists. This still needs to be expanded to cover richer context.

My [Neo4j Graph RAG system](https://github.com/Shashankss1205/neo4j-graph-rag)
can now answer complex queries like “Which file has the most methods?” and
“What design patterns are used in this repo?” — exactly the kind of structural
understanding I envisioned.

# Goal 2: Bitcoin World Knowledge using Open-Source Code Ingestion

**Original Plan** : Ingest written code as primary source and Python-Bitcoin-
Utils, Bitcoin Core, rust-bitcoin, Lightning, LND, libbitcoin as secondary
sources.

**What I’ve Built** : I’ve created the infrastructure to handle this through
my graph-based systems and have begun indexing key Bitcoin repositories. The
architecture supports automatic detection and scraping of Bitcoin-related
repositories, though I’m still expanding the coverage to include all the
repositories I originally targeted.

# Goal 3: Bitcoin World Knowledge using Conversation Ingestion

**Original Plan** : Ingest conversations from IRC, bitcointalk, bitcoin-dev,
Delving Bitcoin, and mailing lists, with potential MCP integration for real-
time data.

**What I’ve Delivered** : This has been one of my mediocre achievements. I’ve
built:

  * [**Bitcoin Mailing List RAG**](https://github.com/Shashankss1205/btc-mail-rag): A comprehensive FAISS-based system that indexes the entire Bitcoin development mailing list archive
  * [**Bitcoin Stack Exchange QnA Generator**](https://github.com/Shashankss1205/btc-mail-rag/tree/main/bitcoin-stack-exchange): Creates question-answer pairs from Bitcoin Stack Exchange for evaluation purposes

# Goal 4: Creating FIM Tool, Code Generation Model, and Chatbot

**Original Plan** : Develop Fill-in-Middle tools, integrate compiler-based
tools, and create a general-purpose chatbot equipped with Bitcoin universe
data.

**Current Status** : I’ve laid the groundwork with my agentic Graph RAG
systems that can understand code structure and provide intelligent responses.
The architecture I’ve built supports the multi-agent approach I originally
envisioned, with separate knowledge graphs for local code and world code. I’ve
actually exceeded my original goal by implementing a Model Context Protocol
server that provides real-time integration capabilities

# Goal 5: Benchmarking and Community Integration

**Original Plan** : Benchmark against existing methods, develop VS Code
extensions, and gather community feedback.

**What I’ve Built** : I’ve developed a comprehensive evaluation framework
using BLEU and BERTScore metrics, testing against both Bitcoin Stack Exchange
data and content from “Grokking Bitcoin.” This provides the rigorous testing
foundation I originally planned. Secondly, as previously stated, I built an
MCP server which can be plugged in any IDE and be tested by the community.

# Technical Deep Dive: Architecture Evolution

My original architecture proposal centered around an agentic LM with three key
tools:

  1. Local Code Knowledge Graph and Traversal Tool
  2. World Code Knowledge Graph and Traversal Tool
  3. World Knowledge Base and Retrieval Tool

I’m proud to say I’ve implemented the first 2 parts of this architecture,
though with some intelligent adaptations:

# The Knowledge Graph Approach

Although I earlier started with ArangoDB, but instead of NetworkX and
ArangoDB, I’ve implemented robust solutions using Neo4j . Through extensive
testing, I’ve confirmed my original hypothesis that Neo4j is superior for this
use case — Cypher queries are indeed better generated by LMs than AQL queries.

Neo4j Graph Database

# The MCP Innovation

One area where I’ve exceeded my original proposal is in MCP (Model Context
Protocol) implementation. I’ve built a comprehensive MCP server that provides:

  * Background processing for large repositories
  * Real-time progress tracking
  * Smart code search with relevance scoring
  * Automatic package discovery

# Key Technical Achievements

WorkFlow Diagram

# 1\. Multi-Database Architecture

I’ve successfully implemented and compared both Neo4j and ArangoDB approaches,
validating my original architectural decisions through empirical testing.

# 2\. Comprehensive Evaluation Framework

My evaluation system using BLEU and BERTScore metrics provides the rigorous
benchmarking foundation I originally envisioned, testing against curated
Bitcoin knowledge sources like Bitcoin-Stack-exchange.

# 3\. Scalable Processing Pipeline

The background job processing system I’ve built can handle large repositories
while providing real-time feedback — solving the scalability challenges I
anticipated in my proposal.

# 4\. Real-World Integration

The MCP server makes these sophisticated AI tools accessible through
development environments, bringing my vision of practical developer assistance
to reality.

# Challenges and Adaptations

## Challenge 1: Scope Management

**Original Challenge** : My proposal was highly ambitious, potentially too
broad for a single project. **Adaptation** : I’ve focused on building solid
foundations first, ensuring each component works excellently before expanding
to the next.

## Challenge 2: Database Selection

**Original Uncertainty** : I proposed testing both Neo4j and ArangoDB to
determine the best fit. **Resolution** : Through implementation and testing,
I’ve validated that Neo4j is indeed superior for this use case, as I suspected
in my proposal.

## Challenge 3: Evaluation Methodology

**Original Plan** : Develop Bitcoin-specific benchmarks. **Implementation** :
I’ve created evaluation frameworks using established metrics (BLEU, BERTScore)
while testing against Bitcoin-specific knowledge sources. I realized that the
testing framework needs even better evaluation strategies.

# Gratitude to My Mentors: A Unique Multi-Organization Collaboration

This project has been uniquely enriched by having mentors from three different
Bitcoin organizations, each bringing distinct perspectives and expertise:

  * **Kostas from python-bitcoin-utils** : Providing deep insights into Bitcoin library development and Python ecosystem best practices
  * **Bob from Braidpool** : Contributing expertise in Bitcoin mining protocols and distributed systems architecture
  * **Andreas from ChatBTC** : Offering guidance on AI/ML applications in Bitcoin and user experience design

What makes this mentorship particularly special is that both Kostas and Bob
had independently accepted me for their respective projects earlier in the
selection process. Recognizing the synergy between their objectives — Bob’s
need for intelligent code analysis tools for Braidpool development and
Kostas’s vision for enhanced python-bitcoin-utils developer experience — the
projects were merged into a single, more comprehensive initiative.

This unique collaboration has resulted in weekly meetings that provide:

  * **Technical Architecture Guidance** : Multi-perspective input on database choices and system design decisions
  * **Cross-Project Insights** : Understanding how tools built can benefit broader Bitcoin development (python-bitcoin-utils, Braidpool, ChatBTC)
  * **Real-World Validation** : Ensuring solutions address actual pain points across different Bitcoin development contexts
  * **Community Integration** : Leveraging connections across multiple organizations to maximize impact

Their combined expertise spanning library development, mining protocols, and
AI applications has shaped this project into something that transcends any
single use case — creating tools that serve the entire Bitcoin development
ecosystem.

# Impact: From Proposal to Practice

The systems I’ve built directly address the core problem I identified in my
proposal: **LMs hallucinating Bitcoin-specific information**. Now, instead of
guessing API parameters or making up function signatures, developers can
access:

  1. **Accurate Code References** : Through graph-based code analysis
  2. **Historical Context** : Via searchable mailing list archives
  3. **Structural Understanding** : Through intelligent code relationship mapping
  4. **Verified Knowledge** : Through rigorous evaluation frameworks

# Looking Ahead: Completing the Vision

As I enter the second half of Summer of Bitcoin 2025, I’m focused on
completing the remaining goals from my original proposal:

# Next Priorities

  1. **SLM Performance** : Achieving the Small Language Model capabilities I originally envisioned
  2. **Community Testing** : Gathering feedback from Bitcoin developers as planned
  3. **Enhanced Multi-Modal Capabilities** : Extending beyond my original text-focused approach
  4. **Real-Time Community Integration** : Leveraging the MCP infrastructure for live developer assistance
  5. **Advanced Evaluation Metrics** : Developing Bitcoin-specific testing criteria

# Technical Learnings: Beyond the Proposal

This project has taught me lessons that go beyond my original scope:

  * **The importance of rigorous evaluation** : My testing frameworks will prove essential for maintaining quality
  * **The value of incremental development** : Building solid foundations before expanding has prevented architectural debt
  * **The power of community knowledge** : The mailing list RAG system has revealed insights I didn’t anticipate in my proposal

# Open Source Commitment

True to the spirit of my original proposal, everything I’ve built is open
source and available to the community. The evaluation frameworks, graph-based
systems, and MCP server can all be used by other projects building Bitcoin
development tools.

# Conclusion: Building the Future I Envisioned

With strong foundations now in place and the continued support of my
incredible mentors, the remaining weeks of Summer of Bitcoin 2025 will see the
completion of my original vision: a comprehensive AI assistant that truly
understands Bitcoin development and empowers developers at every level.

# Links to the Code work:

1\. <https://github.com/Shashankss1205/btc-mail-rag/tree/main>  
2\.
[https://github.com/Shashankss1205/GraphRAG/](https://github.com/Shashankss1205/GraphRAG/blob/main/GRAPHRAG.ipynb)  
3\. <https://github.com/Shashankss1205/neo4j-graph-rag>  
4\. <https://github.com/braidpool/GraphRAG>

_This mid-term evaluation represents approximately 60% completion of my
original proposal goals, with strong foundations in place for achieving the
remaining 40% in the coming weeks._


