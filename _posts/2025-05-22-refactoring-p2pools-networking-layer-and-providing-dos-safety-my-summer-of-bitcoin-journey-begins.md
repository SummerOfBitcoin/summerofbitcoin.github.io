---
layout: post
title: "Refactoring P2Pool’s Networking Layer and providing DoS safety— My Summer of Bitcoin Journey Begins"
date: 2025-05-22
author: Raunak Kumar
categories: [P2Pool, Stories]
---

Hi there! I’m Raunak a first year cs undergrad and a contributor in **Summer
of Bitcoin 2025** , working on an exciting and technically rich project:
enhancing the **P2Pool v2** decentralized mining protocol.

P2Pool v2 operates a **share chain** — a blockchain of mining shares created
by miners, complete with **Uncle blocks** , a **UTXO-based transaction
engine** , and **Bitcoin-like P2P protocols** for node communication. Think of
it as building a mini version of Bitcoin, purpose-built for decentralized pool
mining.

My project focuses on a critical layer of this infrastructure: **protecting
the P2P network from DoS (Denial of Service) attacks**. To achieve this, I’m:

  * Studying how Bitcoin Core defends against various DoS threats,
  * Evaluating which protections are applicable to P2Pool,
  * And most importantly, **refactoring the networking layer to use**`**Tokio Tower**`, which offers better abstractions for managing connections, applying rate limits, timeouts, peer scoring, and more.

In this blog series, I’ll share weekly updates on what I’m working on, what
I’ve learned, and where we’re headed next. This first post captures the
**foundation and refactoring work** I tackled in **Week 1**(15–5–2025)

# What I Set Out to Do

This week, my primary goal was to **lay the groundwork for refactoring P2Pool
v2’s networking stack** to enable future DoS protection features.

Specifically, I planned to:

  * **Understand the existing REQ/REP logic** used in the current `libp2p::request_response` protocol.
  * Explore **how Bitcoin handles DoS attacks** — including connection limits, timeouts, peer scoring, and rate limiting.
  * Learn how **Tokio Tower’s**`**Service**`**trait** can abstract P2P message handling in a modular and testable way.
  * Design a modular layout for the new network stack, separating the concerns of decoding, routing, and responding to messages.
  * Begin implementing a basic Tower `Service` (`P2PoolService`) capable of handling common request types like `Ping`, `Pong`, and `Share`.

The idea was to decouple message logic from libp2p internals and create an
extensible platform for applying DoS protections later — including per-peer
throttling, message validation layers, and behavior-based bans.

# What I Did

Over the first week, I focused heavily on understanding the existing network
flow and setting up the foundation for the Tower-based refactor.

Here’s what I accomplished:

  * **Explored and traced the legacy**`**handle_request_response**`**logic** in the `libp2p::request_response` handler. This gave me insight into how P2Pool currently matches and responds to messages from peers.
  * **Broke down the codebase into more modular components** , moving the network logic into separate files like `server.rs`, `request_response_handler.rs`, and `p2p_message_handlers/mod.rs`.
  * **Began implementing a**`**P2PoolService**`**struct** , which wraps the core request-handling logic into a `tower::Service<Request>` trait implementation. This makes it possible to plug in middlewares like rate limiting, timeouts, or peer bans.
  * **Reorganized message matching** (e.g. for `Ping`, `Pong`, `Share`) into this Tower service so it can be tested independently of libp2p.
  * Started thinking about how to **bridge libp2p’s event-based API** with Tower’s `poll_ready` \+ `call` based model using async wrappers like `BoxFuture`.

While this refactor doesn’t add DoS protection yet, it clears the path for it.
With this Tower-based architecture, we’ll soon be able to treat things like
peer reputation, rate limits, and connection filters as pluggable layers.

# Challenges Faced

As expected with any foundational refactor, there were some tricky parts this
week:

  * **Bridging libp2p with Tower:**  
Libp2p uses an event-driven model, while Tower expects a more service-oriented
request/response flow. Translating incoming messages into
`tower::Service::call` invocations in a clean and future-proof way required
some design iteration.

  * **Retaining existing behavior:**  
I had to ensure that basic P2P behavior (like ping-pong liveness checks and
share propagation) continued to work during the transition. Preserving
constants like `AGENT_VERSION` and `PROTOCOL` while swapping out the internal
handler took careful testing.

  * **Managing async complexity:**  
Working with Rust’s async ecosystem (libp2p, tokio, tower, futures) means
dealing with lots of lifetimes, traits, and pinned futures. Writing a clean
Tower `Service` that compiles _and_ fits into the event loop wasn’t always
straightforward.

  * **Finding the right file/module structure:**  
Since the existing P2Pool code wasn’t originally modular, I spent time
figuring out a sensible directory layout that separates protocol definitions,
handler logic, and service plumbing without introducing circular dependencies.

These challenges slowed me down a bit, but they also gave me a much clearer
understanding of the internals — which will help a lot in the coming weeks.