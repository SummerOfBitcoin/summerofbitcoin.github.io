---
layout: post
title: "Building Confidence in Jam: My Final Summer of Bitcoin Report"
date: 2026-08-22
author: Parth Bandwal
categories: [Jam, Wallets, Privacy, Development, Open-Source]
image: ../assets/images/blog_content/2026-06-27-parth-bandwal-sob-jam-journey.png
---

At the midterm, I wrote about returning to Jam after not being selected for
Summer of Bitcoin the previous year. This final report is about what happened
after that return became a real responsibility: building confidence in a
Bitcoin privacy wallet through tests, safer workflows, clearer user feedback,
and careful review.

My project was **Comprehensive Test Coverage for the Core Send and Earn Flows,
with CI Coverage Enforcement**. The original scope was testing, but the test
work gave me a broad understanding of Jam. After completing that scope, I used
the remaining program time to improve production flows across the application.

## What I Set Out To Do

Jam is a web interface for JoinMarket. Its Send and Earn flows sit close to
wallet state, UTXOs, addresses, offers, and transactions. A regression in one
of these areas can be more than a visual inconvenience, so the project needed
tests that described user-visible behavior and failed reliably in CI.

I set out to:

- add meaningful automated coverage for the core Send and Earn workflows;
- cover surrounding wallet-critical components, hooks, stores, and utilities;
- test loading, error, empty, and interaction states instead of only happy
  paths;
- stabilize timing-sensitive tests and mocks; and
- enforce coverage thresholds in CI so future changes could not silently
  remove tested behavior.

## Building The Test Foundation

The first milestone focused on shared code. [PR
#1267](https://github.com/joinmarket-webui/jam/pull/1267) added tests for
utilities and reusable components, including behavior around copying values,
formatting data, and common UI interactions. [PR
#1270](https://github.com/joinmarket-webui/jam/pull/1270) then made the CI
coverage upload resilient by skipping it when an earlier failure prevented the
coverage report from being generated. This removed a misleading secondary
error and kept the original failure visible.

The main testing milestone was [PR
#1276](https://github.com/joinmarket-webui/jam/pull/1276). It touched 106 files
while expanding tests across components, hooks, context providers, wallet
flows, settings, onboarding, orderbook behavior, and shared state. At that
merged checkpoint, combined unit and Storybook coverage reached **90.6% for
statements, 90.7% for lines, 89.1% for functions, and 82.7% for branches**.
The PR also established 80% thresholds for statements, lines, functions, and
branches.

The threshold was not the goal by itself. It was a guardrail. The useful part
was identifying behavior that the application depended on and leaving behind
a test that would explain when that behavior changed.

## Major Technical And Design Decisions

### Test Behavior, Not Implementation Details

Tests are easier to maintain when they describe what a user or caller can
observe. I focused on questions such as:

- Does an error state explain what failed?
- Is an action disabled while wallet state makes it unsafe?
- Does a dialog reset after it closes?
- Does a loading state prevent a duplicate action?
- Does a route remain stable when authentication or developer mode changes?

This approach made the tests useful during later refactors because the
implementation could change while the expected behavior stayed clear.

### Standardize Validation At The Boundary

Send, Receive, Sweep, and Fidelity Bond forms had related validation rules but
did not always express them in the same place. [PR
#1315](https://github.com/joinmarket-webui/jam/pull/1315) introduced shared
validators for source jars and Bitcoin addresses. It also aligned address,
network, reused-address, and amount checks across flows.

[PR #1357](https://github.com/joinmarket-webui/jam/pull/1357) moved jar
selection to React Hook Form and Yup, and [PR
#1358](https://github.com/joinmarket-webui/jam/pull/1358) applied the same
form model to the remaining dialog flows. The important decision was not the
library choice alone. It was ensuring that selection remained valid at submit
time, even if wallet state changed after the dialog opened.

### Reuse Transaction Logic Where The Risk Is Shared

[PR #1345](https://github.com/joinmarket-webui/jam/pull/1345) reworked the
Fidelity Bond creation, renewal, and move-to-jar experiences around shared
dialog components and hooks. The renewal and move flows now share the sequence
that freezes other UTXOs, unfreezes the bond, sends it, and restores the prior
frozen state. Keeping that sequence in one place reduced duplicated
transaction logic and made error handling consistent.

The same PR aligned step progress, jar selection, confirmation, success
details, mobile layout, and footer actions across all three dialogs. This was
a design and reliability improvement together: consistent interfaces are
easier for users to understand and easier for maintainers to test.

### Make Important State Visible

Several later changes followed one principle: do not make users infer wallet
or network state when the application already knows it.

- [PR #1367](https://github.com/joinmarket-webui/jam/pull/1367) blocks new
  receive-address requests while a wallet rescan is active.
- [PR #1408](https://github.com/joinmarket-webui/jam/pull/1408) prevents users
  from entering Fidelity Bond creation without an eligible UTXO and explains
  how to obtain one.
- [PR #1420](https://github.com/joinmarket-webui/jam/pull/1420) shows whether a
  maker's offer appears in the local orderbook, including checking, visible,
  not-found, and error states.
- [PR #1433](https://github.com/joinmarket-webui/jam/pull/1433) replaces
  arbitrary fee ranges with JoinMarket NG quantization bands, provides relative
  and absolute views, and adds accessible explanations for exact and rounded
  fees.
- [PR #1441](https://github.com/joinmarket-webui/jam/pull/1441) creates the
  router once and moves state-dependent decisions into reactive guards, keeping
  route configuration stable as application state changes.

## What Shipped

The agreed project scope is complete and merged. The final result includes:

- broad automated coverage for Send, Earn, and surrounding wallet-critical
  behavior;
- CI coverage enforcement across all four reported metrics;
- more stable asynchronous tests and clearer CI failures;
- shared form validation and standardized dialog form handling;
- unified Fidelity Bond workflows and responsive dialog behavior;
- safer receive-address behavior during rescans;
- clearer Fidelity Bond eligibility feedback;
- local maker-offer visibility and fee quantization views; and
- stable route configuration with regression coverage.

I also reviewed contributor PRs, investigated CI failures, discussed
implementation choices, and helped work through review feedback. Those tasks
were not a separate deliverable, but they were an important part of learning
how a maintained open-source project moves forward.

## What Remains

Nothing from the agreed primary testing scope remains unfinished or unmerged.

At the final report checkpoint on August 22, 2026, one additional security
improvement, [PR #1450](https://github.com/joinmarket-webui/jam/pull/1450),
was under review. It removes a cached seed phrase when password
verification expires, resets the reveal state after re-verification, and adds
regression coverage for that timeout behavior. It is follow-up work beyond the
original project scope.

## Mentorship And Ownership

I owned the testing project from identifying coverage gaps through
implementation, debugging, CI stabilization, review updates, and integration.
As my understanding of the codebase grew, I also proposed and implemented
production changes across wallet flows, validation, routing, orderbook views,
and Fidelity Bond experiences.

My mentor, [tbk](https://github.com/theborakompanioni), gave me substantial
freedom while remaining available whenever I needed JoinMarket context,
architectural direction, or a rigorous review. He did more than suggest
changes: he explained the risks behind them, questioned assumptions, and
pushed me to reason about edge cases and long-term maintainability. His
guidance improved both my engineering judgment and my confidence as an
open-source contributor.

## What I Learned

The largest lesson was that tests are part of product design. A good test asks
what the application promises to the user and records that promise in a form
that future contributors can run.

I also learned that a coverage percentage needs context. High coverage can
still miss the state transition that matters, while one focused regression
test can protect a critical workflow. Thresholds are useful because they stop
coverage from quietly moving backward, but thoughtful cases are what make the
suite valuable.

Working on Jam also changed how I approach frontend engineering. Wallet state,
privacy, and transaction behavior make seemingly small UI decisions important.
A disabled button, a visible error, a stable route, or a restored frozen state
can be part of protecting the user.

Finally, I learned what ownership in open source looks like: understand the
problem, propose a direction, accept detailed review, revise the patch, and
remain responsible for how the change fits the rest of the project.

## What Happens Next

Summer of Bitcoin ends, but my work with Jam does not. I plan to continue
contributing to the project, reviewing changes where I have context, and
improving wallet reliability, security, and maintainability. I also want to
carry the same habits into future Bitcoin work: verify assumptions, make
important state visible, and leave behavior better protected than I found it.

For anyone interested in Bitcoin open source, the best starting point is not
waiting until you feel fully ready. Read the code, run the tests, follow review
conversations, and begin with one concrete problem. My own journey started
with rejection and continued because I came back and kept contributing.

Thank you to Summer of Bitcoin, tbk, the Jam maintainers, and every contributor
whose review or discussion helped me improve this work.
