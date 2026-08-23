---
layout: post
title: "From Bitcoin Data to Live Mining: Building the Braidpool Dashboard"
date: 2026-08-23
author: Priya Rani
categories: [Mining, Braidpool, Open-Source, Stories]
image: ../assets/images/blog_content/2026-08-23-braidpool-logo.png
---

Nobody warns you that a dashboard can lie. Not maliciously, just quietly, by design, because someone needed *something* on screen and a simulator was the fastest way to get there. I spent the better part of this summer fixing that lie.

I'm Priya, and this is my Summer of Bitcoin 2026 final report. I worked on [Braidpool](https://github.com/braidpool/braidpool), a decentralized Bitcoin mining pool where hashrate is pooled without trusting a single coordinator. My job, officially, was to improve the dashboard. Unofficially, it turned out to be something closer to: *figure out why nothing on the screen reflects what the node is actually doing, and then fix it.*

---

## A little context on Braidpool

Most mining pools today are centralized. You point your ASIC at a pool server, the server decides who gets paid and how much, and you trust them. Braidpool takes a different approach: it uses a DAG (directed acyclic graph) of cryptographically linked work shares (called "beads") to distribute payouts trustlessly. The node itself is Rust all the way down.

The dashboard is what node operators use to see the state of their node: the bead DAG growing in real time, which miners are connected, transaction status, network health. At least, that's the idea. When I joined, it was more like a well-designed mockup that happened to run in a browser.

---

## Week one reality check

I applied to this project thinking it was a frontend problem. React, TypeScript, some charts, all territory I knew. The first week I spent reading code cured me of that assumption pretty fast.

The React app existed. The Rust backend had WebSocket RPC infrastructure. But the two were barely talking to each other. The DAG visualization ran off a local simulator generating fake bead chains. The miner management section had function stubs but no persistence, no discovery, no actual data. The transactions view was an empty table.

There's a specific kind of uncomfortable feeling when you realize the thing you're looking at isn't real. Like finding out the "live" demo you watched in a product video was pre-recorded. That feeling is what drove most of my decisions for the next three months.

Every frontend fix I wanted to make ran into the same wall: *there's no API for this.* So I stopped thinking of my work as frontend and started thinking of it differently: start from the data, build backward to the screen.

---

## The foundation work (early PRs)

Before the bigger features, I spent the first part of the program building the groundwork, the things that had to exist before anything else could.

I started outside the main repo entirely. [PR #3 on rust_cpunet_miner](https://github.com/braidpool/rust_cpunet_miner/pull/3) added an Axum-based HTTP API for miner statistics and health checks, with CORS support so the dashboard could actually talk to it. Small thing, but it was my first real piece of Rust in this project and it established the pattern I'd follow everywhere: build the API first, wire the frontend second.

Back in the main repo, [PR #406](https://github.com/braidpool/braidpool/pull/406) improved the braid visualisation, moving it to a proper Bead Explorer view and fixing the zoom behaviour, which had been driving me slightly mad. Then [PR #427](https://github.com/braidpool/braidpool/pull/427) integrated real Braidpool node data into the dashboard through WebSocket RPC for the first time. That PR felt like a turning point. Before it, the dashboard was self-contained and fake. After it, it was actually connected to something real.

[PR #469](https://github.com/braidpool/braidpool/pull/469) removed unused dependencies. Boring work that nobody notices until a build breaks because of something you didn't mean to include. [PR #470](https://github.com/braidpool/braidpool/pull/470) added ARIA labels to the download buttons. Accessibility work rarely gets celebrated, but it should; a tool that only works for some people is a tool that doesn't fully work.

[PR #471](https://github.com/braidpool/braidpool/pull/471) added database-backed miner management with CRUD operations and periodic data refresh, the first real version of the miner management system that would later grow into PR #527.

All of these merged. They're not the flashiest PRs in the list, but they're why the later work was possible.

---

## The DAG: tearing out the simulator

The biggest single thing I did was throw out the DAG simulator.

The simulator was genuinely well-built. It generated realistic bead chains with proper parent relationships, timing, and structure. It was probably the right call when the node's RPC wasn't ready. But it had become a trap. The frontend worked fine in isolation, and nobody had pushed to actually connect it to the node.

I wired the DAG visualization to the node's WebSocket JSON-RPC directly. The frontend now subscribes to bead events and updates the graph incrementally as beads arrive. I added reconnection logic so the dashboard doesn't go blank if the node restarts, it re-subscribes automatically once the connection comes back.

The part that took the most iteration was the subscription race condition. Braidpool's RPC pushes events over the same connection you use to subscribe. Between the moment you send the subscription request and the moment the server confirms it, new beads can arrive. If you're not careful, those beads fall into a gap and your initial state is stale before you even render it. Getting that handoff right, so you never miss a bead and never double-count one, was probably three days of work that isn't visible anywhere in the PR.

**[PR #519: Live DAG WebSocket RPC](https://github.com/braidpool/braidpool/pull/519)** *(Open / under review)*

---

## Transactions: building the API that should have existed

The transactions view had columns and rows and a search bar. It had no data. There was simply no backend method that served transaction state.

I built a set of custom JSON-RPC methods covering the full transaction lifecycle: from first appearance in the mempool through confirmation in a committed bead to final settlement. Since committed transactions weren't indexed for fast lookup in the current node architecture, I added an index. Without it, the RPC would have had to do full scans on every query, which isn't something you want firing on every dashboard refresh.

I kept the API design close to Braidpool's existing RPC conventions: composable methods, consistent error shapes, useful both for the dashboard and for anyone else building tooling on top of the node.

**[PR #515: Transaction lifecycle RPCs](https://github.com/braidpool/braidpool/pull/515)** *(Open / under review)*

---

## ASIC discovery: the detour that was worth taking

My original proposal said to build miner discovery using Pyasic, a Python library with solid ASIC coverage. Reasonable choice on paper. Wrong choice once I was actually inside the codebase.

Braidpool is a Rust project. Everything runs in one process, one binary. Adding Pyasic would have meant a Python sidecar process, some form of IPC, a second runtime to deploy and maintain, and a dependency that has nothing to do with how the rest of the node is built. Three weeks in, I found `asic-rs`, a Rust crate the project was already pointing toward for exactly this purpose.

I scrapped the Pyasic plan and built `miner-api-rs` instead: a Rust wrapper around `asic-rs` that adds automatic LAN subnet scanning. It walks the local network, finds ASICs, and reports their status back to the node. No Python, no sidecar, no IPC. Just a library that fits.

My mentor agreed. I shipped it. It merged.

That pivot taught me something I'll carry into every project I work on: *reading the codebase before writing any code is not optional.*

**[PR #513: Miner API & ASIC discovery](https://github.com/braidpool/braidpool/pull/513)** ✅ **Merged**

---

## Miners: from discovery to full lifecycle management

Knowing a miner exists on your LAN is one thing. Tracking it across reboots, IP changes, and restarts is another problem entirely.

I built the full miner management stack: CRUD APIs, SQLite persistence, a background polling loop for telemetry, and WebSocket pushes so the dashboard gets live status without hammering the API. Simple enough on the surface. But there was a real design question buried in it.

ASICs don't have static IP addresses. They get new DHCP leases. You move one to a different switch and it comes up with a different address. If your miner identity is tied to an IP, your database gets a new record every time that happens , a miner you've been tracking for weeks becomes an unknown stranger. I caught this early ([Issue #485](https://github.com/braidpool/braidpool/issues/485)), documented it, and switched to MAC addresses as the stable identifier. Small decision, real consequence.

**[PR #527: Miner management & monitoring](https://github.com/braidpool/braidpool/pull/527)** *(Open / under review)*

---

## Node Health: more signal, less noise

The Node Health page showed basic status. I redesigned it around what an operator actually needs to know quickly: real-time inbound and outbound bandwidth, with selectable time ranges so you can see whether a spike happened in the last ten minutes or has been building for hours. All live, over WebSocket, no page refresh.

The visual changes look simple. The underlying work was adding the right RPC methods on the Rust side to actually expose that data in a usable shape.

**[PR #520: Node Health redesign](https://github.com/braidpool/braidpool/pull/520)** *(Open / under review)*

---

## The quieter work

Two smaller PRs that don't make headlines but matter:

- **[PR #529](https://github.com/braidpool/braidpool/pull/529):** Tower/Tower-HTTP middleware integration and transaction-fetching cleanup. The kind of refactor that makes the next feature easier to write.
- **[PR #521](https://github.com/braidpool/braidpool/pull/521):** Replaced a non-standard `ActionIconButton` with the project's standard button component for download functionality. Tiny. But consistency in a shared codebase compounds.

---

## Issues I filed when I couldn't fix things yet

Not everything I noticed was mine to fix immediately. Some things needed discussion first. Some were genuinely good first issues for other contributors. I filed them anyway.

| Issue | What it's about |
|-------|-----------------|
| [#459](https://github.com/braidpool/braidpool/issues/459) | Proposed connecting the frontend directly to the Rust WebSocket RPC, the architectural idea behind most of the live-data work |
| [#460](https://github.com/braidpool/braidpool/issues/460) | Planning issue tracking all dashboard monitoring improvements |
| [#485](https://github.com/braidpool/braidpool/issues/485) | Duplicate miner records on IP change (miner identity problem) |
| [#499](https://github.com/braidpool/braidpool/issues/499) | RPC method to fetch cumulative work for a specific bead |
| [#500](https://github.com/braidpool/braidpool/issues/500) | DAG visualization UI should use Tailwind instead of ad-hoc styles |
| [#501](https://github.com/braidpool/braidpool/issues/501) | Integrate miner discovery deeper into the node itself |
| [#507](https://github.com/braidpool/braidpool/issues/507) | DAG was recomputing the Highest Work Path on every single update, which is unnecessary |
| [#514](https://github.com/braidpool/braidpool/issues/514) | Vite CJS Node API deprecation warning, proposed moving to ESM config |

Issue #459 was the one that mattered most strategically. I opened it on May 9 before writing a single line of new code, because I wanted the maintainers to agree on the direction before I went too far down any path. That conversation shaped the next three months.

---

## On ownership

I want to say this plainly, because I think it gets hedged too much in these reports.

The implementation, every line of Rust and TypeScript in those PRs, every design decision, every bug I found and fixed before review, every issue I opened: that was my work. I ran the day-to-day problem solving without hand-holding.

My mentor and the maintainers gave me something different and equally valuable: architectural instincts I didn't have yet. When I was about to build the Pyasic integration, my mentor helped me see why `asic-rs` was the right call. When my PR touched something sensitive in the node's RPC layer, the reviewers caught what I couldn't see from the outside. I didn't pretend to not need that. I asked for it.

Good open-source mentorship isn't about someone building things for you. It's about having people around who've made the mistakes you haven't made yet.

---

## Full artifact list

| PR | Title | Status |
|----|-------|--------|
| [rust_cpunet_miner #3](https://github.com/braidpool/rust_cpunet_miner/pull/3) | CPUNet Miner HTTP API | ✅ Merged |
| [#406](https://github.com/braidpool/braidpool/pull/406) | Braid visualisation & Bead Explorer | ✅ Merged |
| [#427](https://github.com/braidpool/braidpool/pull/427) | Node RPC integration | ✅ Merged |
| [#469](https://github.com/braidpool/braidpool/pull/469) | Dependency cleanup | ✅ Merged |
| [#470](https://github.com/braidpool/braidpool/pull/470) | Dashboard accessibility (ARIA labels) | ✅ Merged |
| [#471](https://github.com/braidpool/braidpool/pull/471) | Database-backed miner management | ✅ Merged |
| [#513](https://github.com/braidpool/braidpool/pull/513) | Miner API & ASIC discovery | ✅ Merged |
| [#515](https://github.com/braidpool/braidpool/pull/515) | Transaction lifecycle RPCs & indexing | Under review |
| [#519](https://github.com/braidpool/braidpool/pull/519) | Live DAG via WebSocket RPC | Under review |
| [#520](https://github.com/braidpool/braidpool/pull/520) | Node Health improvements | Under review |
| [#521](https://github.com/braidpool/braidpool/pull/521) | Download UI refactor | ✅ Merged |
| [#527](https://github.com/braidpool/braidpool/pull/527) | Miner management & monitoring | Under review |
| [#529](https://github.com/braidpool/braidpool/pull/529) | RPC/backend improvements | Under review |

---

## What this summer actually taught me

I came in thinking the hard part would be the React code. The hard part was Rust. Not the syntax; that's just reading. The hard part was understanding the *shape* of a production Rust codebase: where state lives, how errors are meant to propagate, what a PR that fits looks like versus one that technically works but fights the conventions. That took weeks of reading before it clicked.

I also learned that filing an issue is not admitting defeat. Before this, I would have either silently fixed something or ignored it. Now I understand that documented problems with good reproduction steps are genuinely valuable, sometimes more valuable than a half-finished fix. A few of those issues have already attracted other contributors. That wasn't something I expected to be proud of, but I am.

---

## One last thing

There's still a lot of empty space in this dashboard. Some PRs are still in review. More features are planned that I didn't get to. The codebase is young and fast-moving and I'll probably look back at some of my code in a year and wince.

But here's the thing I keep coming back to: when you open the Braidpool dashboard today and look at the DAG, you're watching your actual node. The beads are real. The miner status is real. The transaction history is real. None of it is simulated anymore.

A dashboard that shows you the truth about your system is not a small thing. That's what I came here to build, and that's what I built.

Priya
