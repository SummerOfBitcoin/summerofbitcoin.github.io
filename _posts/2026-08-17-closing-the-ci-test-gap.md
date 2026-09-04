---
layout: post
title: "Closing the CI Test Gap: My Summer of Bitcoin Journey with Shopstr"
date: 2026-08-17
author: Arnav Kirti
categories: [Shopstr, Nostr, Stories, Development]
image: ../assets/images/blog_content/2026-08-17-coverage.png
---

I’m Arnav Kirti, a B.Tech student at IIT Roorkee and a full-stack developer. I enjoy working across different parts of the stack and, more importantly, contributing to open-source projects where the code I write has to work for real users.

For Summer of Bitcoin 2026, I had the opportunity to work with Shopstr, a decentralized marketplace built around Bitcoin, Nostr, and Cashu. What initially started as a focused project around improving Shopstr’s testing infrastructure turned into a much broader exploration of production-grade open-source development.

---

## What I Set Out to Do

When I started the project, Shopstr already had a substantial Jest test suite, but there was an important gap: tests were not being enforced on pull requests. The full suite primarily ran during release workflows, meaning regressions in critical areas could make it into the main branch without being caught early.

My initial goal was therefore straightforward:

> Make testing a first-class part of Shopstr’s development workflow.

The plan was to introduce PR-level CI testing, improve coverage around high-risk Nostr, database, payment, and marketplace flows, make tests deterministic, and establish clear coverage expectations for future contributors.

The original proposal focused heavily on closing this CI gap and protecting the parts of Shopstr where failures could directly affect marketplace and payment state.

---

## What I Built

The project ended up going significantly beyond the original scope.

### CI and Testing Infrastructure

I added and maintained a dedicated GitHub Actions test workflow that runs:

* Jest coverage tests
* Testcontainers-based integration tests

The workflow runs on pull requests and pushes to main, uses Node 22 and npm caching, and includes concurrency controls to avoid unnecessary CI runs.

I also updated Jest’s coverage configuration to include the mcp/** codebase, making MCP functionality part of the visible coverage surface rather than leaving it effectively invisible to coverage reporting.

### A Much Larger Test Surface

Testing expanded across:

* MCP read, write, and purchase tools
* MCP authentication and authorization
* MCP API routes
* Nostr helpers and relay behavior
* Product and community parsers
* Cashu and Lightning payment flows
* Database and cache helpers
* Checkout components
* API handlers
* Utility functions

The scale of the test suite changed considerably during the project. The repository went from roughly 131 to 164 test files, while test declarations increased from roughly 1,564 to 2,580.

A verified Jest run reached 163 test suites and 2,561 tests.

More importantly, overall coverage improved from:

| Metric | Before | After |
|---|---:|---:|
| Statements | 52.1% | 66.28% |
| Branches | 39.4% | 56.36% |
| Functions | 49.8% | 60.64% |
| Lines | 52.5% | 67.02% |

The goal wasn’t simply to increase a number. I focused on making the coverage meaningful by targeting code involved in trust, money movement, state transitions, and protocol handling.

---

## Major Technical Work Beyond the Original Scope

As I worked through the codebase, testing often exposed opportunities to improve the underlying implementation itself.

### MCP Improvements

The MCP work grew substantially beyond writing tests. I worked on:

* Shared seller caching
* Pagination cursor helpers
* Category filtering and registry support
* Search improvements
* Rate limiting
* `get_categories`
* Seller and storefront tool cleanup
* Read/write/purchase authorization paths

This made the MCP layer not only better tested, but also cleaner and more capable.

### Nostr and Relay Hardening

I also worked extensively around Nostr-related behavior, including relay metadata and NIP-50 search behavior.

Several protocol-sensitive areas received additional regression coverage, including request authentication, signer behavior, gift-wrap handling, reports, zap validation, tags, filters, and relay interactions.

These were particularly interesting because failures here are rarely isolated to a single function. A small mistake in event handling can propagate through fetching, parsing, caching, and eventually what a user sees.

### Checkout and Payments

Checkout became another major area of work.

I added coverage around payment dispatch, quote validation, server-side repricing, retries, unknown payment states, Cashu/Lightning/NWC flows, and cart cleanup.

But this wasn’t just testing existing behavior. While writing these tests, I found and fixed real edge cases.

One example was the handling of `Added Cost/Pickup` cart items. They could incorrectly be routed through shipping even when the buyer had selected pickup. The regression test now protects this behavior.

I also extracted seller Lightning payout logic into dedicated modules and added explicit handling for pending and unknown payout states, making recovery behavior clearer and removing duplicated payout logic.

---

## Major Technical Decisions

One of the biggest lessons from the project was that **good tests require understanding the architecture first.**

Rather than trying to maximize coverage everywhere, I prioritized flows based on their potential impact:

**Nostr relay → validation → parsing → cache/database → marketplace state → checkout/payment**

These boundaries are where a seemingly small bug can become a user-visible or financially significant problem.

I also preferred regression tests based on real failure modes. Instead of writing tests only to increase coverage, I tried to capture behaviors that should never regress again.

For example:

* Invalid Nostr events should not become trusted state.
* Conflicting relay state should be resolved deterministically.
* Invalid payment states should not incorrectly promote an order.
* Failed or unknown Lightning payouts need explicit recovery behavior.
* Checkout routing must respect the user’s selected fulfillment method.

---

## What Shipped

From PR **#551 through #611**, I merged **30+ PRs** during the post-midterm period.

Across those PRs:

* ~206 files were changed
* ~37,939 lines were added
* ~3,747 lines were removed
* Test files grew from ~131 to ~164
* Test declarations grew from ~1,564 to ~2,580

The repository now contains:

* A dedicated CI test workflow
* Expanded Jest coverage
* MCP coverage as part of the coverage report
* Large new test suites across core application paths
* Improved MCP functionality
* Hardened Nostr and relay behavior
* Stronger checkout and payment regression coverage
* Refactored Lightning payout handling
* Coverage planning and tracking for future contributors

The project therefore ended up being much more than “adding tests.” It became a combination of **testing, debugging, refactoring, and hardening a production codebase.**

---

## What Remains

There is still more that can be done.

The current coverage is substantially better, but there are still areas of Shopstr that would benefit from deeper integration and end-to-end testing. In particular, checkout-critical UX flows and more adversarial testing of Nostr event handling are natural next steps.

The CI environment also has some integration-test constraints around Testcontainers and Docker. These are documented so that environment-related failures are not confused with application regressions.

---

## What I Learned

The biggest thing I learned this summer was how different contributing to a real production codebase is from building a project from scratch.

In a personal project, you control the architecture, the conventions, and the trade-offs. In open source, you have to understand the decisions that already exist, work within them, and make changes that other contributors can maintain.

I also learned that tests are often a way of understanding a system, not just verifying it. While writing tests for Shopstr’s payment, Nostr, and MCP flows, I ended up discovering architectural relationships and edge cases that would have been easy to miss by simply reading the code.

Most importantly, I got to experience what genuinely decentralized development feels like. The team itself was decentralized, contributors had significant ownership, and progress depended heavily on communication, trust, and being able to make decisions independently.

---

## What Happens Next

I would like to continue contributing to Shopstr after Summer of Bitcoin, particularly around its frontend architecture and state management.

There are also natural follow-ups to this project, including lightweight E2E tests for checkout-critical flows and more extensive fuzz/property testing for parsers and Nostr event handling.

The CI and test infrastructure should also continue evolving as the codebase grows. The goal is not to reach a perfect coverage number, but to make sure that important behavior remains protected as new features and contributors are added.

---

## Closing Thoughts

I started Summer of Bitcoin with a relatively simple goal: **close the CI test gap.**

I’m leaving with a much broader understanding of how to work on production Bitcoin and Nostr software, how to reason about testing across distributed systems, and how to contribute effectively to a decentralized open-source team.

The most rewarding part was seeing a project that started as a proposal turn into real changes in a codebase used by others. I’m grateful to my mentors and the Shopstr team for giving me the autonomy to explore, build, break things, fix them, and ultimately contribute something that can continue to be useful after the program ends.