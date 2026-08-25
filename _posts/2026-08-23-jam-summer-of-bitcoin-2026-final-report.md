---
layout: post
title: "Modernizing Jam Web UI for JoinMarket-NG: Summer of Bitcoin 2026 Final Report"
date: 2026-08-23
author: Kishore B
categories: [Jam, JoinMarket-NG, Bitcoin, Open-Source, Development]
image: ../assets/images/blog_content/2026-08-23-kishore-avatar.jpeg
---

As my Summer of Bitcoin 2026 internship comes to a closure, I am excited to share my final project report detailing my work on [Jam Web UI](https://github.com/joinmarket-webui/jam) and [JoinMarket-NG](https://github.com/joinmarket-ng/joinmarket-ng) ecosystem. Over the past twelve weeks, I worked under the guidance of my mentor [TBK](https://github.com/theborakompanioni) to transition Jam from legacy `joinmarket-clientserver` to the next-generation `joinmarket-ng` architecture, modernizing the frontend stack, improving backend API persistence and delivering full feature parity for Bitcoin privacy workflows.

---

## What I Set Out to Do

[Jam](https://github.com/joinmarket-webui/jam) is the web interface for [JoinMarket](https://github.com/JoinMarket-Org/joinmarket-clientserver), enabling Bitcoin users to participate in collaborative CoinJoin transactions, earn yield by providing liquidity as yield generators (makers), manage UTXO mixdepths, and control privacy parameters without touching command-line tools.

Recently, the JoinMarket ecosystem began migrating toward **JoinMarket-NG (`joinmarket-ng`)**, a modernized architecture of the backend daemon (`jmwalletd`) that replaces legacy RPC patterns with clean RESTful OpenAPI specifications, structured configuration models, and improved session handling.

My primary goal for Summer of Bitcoin 2026 was to lead the frontend modernization and backend alignment required for Jam to fully adopt `joinmarket-ng`. Specifically, I set out to:

1. **Containerize and Bridge `joinmarket-ng` Environment**: Build Docker wrapper infrastructure and Docker Compose setups to seamlessly run and orchestrate `joinmarket-ng` backend services alongside Bitcoin Core for development and testing.
2. **Migrate the API Client Layer**: Establish a strongly typed TypeScript API client library ([`joinmarket-ng-api-ts`](https://github.com/joinmarket-webui/joinmarket-ng-api-ts)) generated from the `joinmarket-ng` OpenAPI specification and integrate it into Jam.
3. **Achieve Full Feature Parity**: Implement wallet transaction history, orderbook pre-checks, active send feedback, QR code scanning, and backend status warning indicators.
4. **Fix Backend API Compatibility**: Resolve transaction history tracking gaps, log routing, and configuration mapping issues in the Python backend (`jmwalletd`).
5. **Modernize Frontend Architecture**: Upgrade Jam's tabular UI components across Orderbook, Earn Report, and Wallet screens from TanStack Table v8 to v9.

---

## What I Built

To accomplish these goals, my work spanned the full stack—from Docker containerization and Python backend fixes in `jmwalletd` to TypeScript API code generation and React frontend engineering in Jam.

Key systems and components built during the summer include:

1. **Standalone-NG Docker Stack (`jam-docker`)**:
   - Developed Docker wrappers and Docker Compose setups to run `joinmarket-ng` seamlessly alongside Bitcoin Core in regtest and testnet environments.

2. **TypeScript API Client (`@joinmarket-webui/joinmarket-ng-api-ts` v1.0.0)**:
   - Migrated OpenAPI spec definitions to match `joinmarket-ng` endpoints and published `@joinmarket-webui/joinmarket-ng-api-ts` v1.0.0.
   - Replaced all legacy API contracts and types in Jam with auto-generated, type-safe React Query hooks and mutation abstractions.

3. **Complete Wallet Transaction History (`jam` & `joinmarket-ng`)**:
   - Designed a unified transaction history UI in Jam displaying collaborative CoinJoins, direct send transactions, and on-chain deposits.
   - Built a three-phase persistence lifecycle (**Prepare -> Broadcast -> Finalize**) in `jmwalletd` to record direct sends atomically in history files at broadcast time without data loss.

4. **Orderbook Pre-Checks & Collaborative Send Enhancements**:
   - Built real-time orderbook maker availability pre-checks prior to initiating send and sweep flows, warning users when insufficient makers are available for requested mixdepths.
   - Enhanced active CoinJoin send cards with detailed progress indicators, step cards, and maker activity feedback.

5. **TanStack Table v9 Refactoring**:
   - Upgraded all tables in Jam (Orderbook, Earn Report, UTXO Wallet lists) to TanStack Table v9 using feature configuration schemas.

6. **Configuration & UX Guardrails**:
   - Mapped `max_sweep_fee_change` in `joinmarket-ng` to enforce runtime relative fee tolerance guards during CoinJoin execution.
   - Added QR code webcam scanning to sweep destination inputs, jar color-coded badges, tooltip balance legibility fixes, and backend-specific info modals.

---

## Major Technical and Design Decisions

Below are the key technical decisions made during the project:

### 1. Multi-Process Container Orchestration via `s6-overlay`
*Decision*: Adopted `s6-overlay` process supervision in the new `standalone-jam-ng` Docker wrapper service image, replacing `dinit` used in legacy standalone Docker images.
*Rationale*: `s6-overlay` provides robust multi-process orchestration, service readiness supervision, clean process lifecycle management, structured log management, and built-in `logrotate` support for long-running containerized backend services.

### 2. Three-Phase History Persistence in `jmwalletd`
*Decision*: Implemented a three-phase persistence pattern (**Prepare**, **Broadcast**, **Finalize**) for non-collaborative transactions in the Python backend.
*Rationale*: Previously, direct sends were written to history only after full block confirmation. If a daemon restarted or lost state after broadcast but before block inclusion, history entries were lost. Writing an unconfirmed entry at broadcast time and updating status upon finalization guarantees atomic, zero-data-loss transaction recording.

### 3. Neutral `amount` Schema for Non-Collaborative Transactions
*Decision*: Replaced reliance on `cj_amount` for direct transactions with a neutral `amount` field in transaction records, explicitly setting `cj_amount` and `source_mixdepth` to `null` for non-CoinJoin entries.
*Rationale*: Overloading CoinJoin fields for standard payments confused UIs and led to incorrect earnings and volume metrics. A unified schema ensures clean separation between collaborative privacy mixes and standard on-chain transactions.


---

## What I Shipped

Across the summer, I authored and merged **25 pull requests** and published **1 npm package release** across 4 repositories in the `joinmarket-webui` and `joinmarket-ng` organizations.

### 1. [joinmarket-webui/jam-docker](https://github.com/joinmarket-webui/jam-docker)

| PR | What it does |
|---|---|
| [**#186** — feat: standalone-ng docker wrapper for joinmarket-ng backend](https://github.com/joinmarket-webui/jam-docker/pull/186) | Creates Docker wrapper scripts and Docker Compose configuration to run `joinmarket-ng` as a standalone backend service stack alongside Bitcoin Core. |
| [**#189** — Add backend info endpoint](https://github.com/joinmarket-webui/jam-docker/pull/189) | Adds a dedicated metadata endpoint to expose backend daemon type, version info, and container health parameters to host UIs. |
| [**#205** — fix: redirect jmwalletd stderr to stdout to capture operational logs](https://github.com/joinmarket-webui/jam-docker/pull/205) | Redirects `jmwalletd` standard error output to standard output within Docker containers, enabling unified log aggregation and troubleshooting. |

### 2. [joinmarket-webui/jam](https://github.com/joinmarket-webui/jam)

| PR | What it solves |
|---|---|
| [**#1265** — feat: integrate standalone-ng docker wrapper with Jam Web UI](https://github.com/joinmarket-webui/jam/pull/1265) | Connects the standalone `joinmarket-ng` Docker wrapper with Jam's local dev workflows and NPM scripts. |
| [**#1281** — Make joinmarket-ng as the default primary backend in regtest env](https://github.com/joinmarket-webui/jam/pull/1281) | Configures `joinmarket-ng` as the primary default backend daemon in regtest local development mode. |
| [**#1282** — feat(sweep): add QR code scanner to destination inputs](https://github.com/joinmarket-webui/jam/pull/1282) | Integrates a webcam QR code scanner into sweep destination address fields for effortless input. |
| [**#1293** — Add tooltip for jm:autofrozen:reuse label](https://github.com/joinmarket-webui/jam/pull/1293) | Adds explanatory tooltips for `jm:autofrozen:reuse` labels to inform users why specific UTXOs were frozen by address reuse rules. |
| [**#1305** — refactor: migrate API client to joinmarket-ng-api-ts](https://github.com/joinmarket-webui/jam/pull/1305) | Replaces legacy API calls in Jam with the new `@joinmarket-webui/joinmarket-ng-api-ts` SDK. |
| [**#1308** — style jar badges with their respective jar colors](https://github.com/joinmarket-webui/jam/pull/1308) | Updates UI jar badges across wallet views to visually distinguish mixdepth jars using distinct color schemes. |
| [**#1340** — docs: replace joinmarket-clientserver references to joinmarket-ng](https://github.com/joinmarket-webui/jam/pull/1340) | Updates documentation across the codebase to systematically reference `joinmarket-ng` instead of legacy `joinmarket-clientserver`. |
| [**#1341** — feat(ui): Show backend-specific warnings in onboarding and footer info modals](https://github.com/joinmarket-webui/jam/pull/1341) | Adds contextual warning banners in onboarding and footer modals detailing active backend daemon capabilities. |
| [**#1368** — feat(wallet): implement wallet transaction history](https://github.com/joinmarket-webui/jam/pull/1368) | Implements the full wallet transaction history UI supporting collaborative CoinJoins, direct send payments, and deposits. |
| [**#1391** — refactor(state): centralize session state management into JamSessionInfoContext](https://github.com/joinmarket-webui/jam/pull/1391) | Centralizes application session state into React `JamSessionInfoContext` and exports `useJamSession` helper hook. |
| [**#1402** — refactor(ui): consolidate Send and Sweep precondition alerts](https://github.com/joinmarket-webui/jam/pull/1402) | Refactors precondition validation banners across send and sweep views into reusable alert components. |
| [**#1406** — refactor: externalize hardcoded polling intervals and delays](https://github.com/joinmarket-webui/jam/pull/1406) | Extracts hardcoded polling intervals into central configuration constants for modular state management. |
| [**#1407** — feat(orderbook): add orderbook availability pre-check for send and sweep](https://github.com/joinmarket-webui/jam/pull/1407) | Adds real-time orderbook maker availability pre-checks prior to starting send or sweep privacy operations. |
| [**#1413** — ui(send): improve active collaborative send info](https://github.com/joinmarket-webui/jam/pull/1413) | Enhances active CoinJoin transaction status cards, step progress indicators, and maker activity feedback. |
| [**#1432** — refactor: Migrate table components from TanStack Table v8 to v9](https://github.com/joinmarket-webui/jam/pull/1432) | Upgrades all tabular UIs (Orderbook, Earn Report, Wallet tables) to TanStack Table v9 with strict TypeScript schemas. |
| [**#1440** — feat(settings): add Sign Message to allow signing messages with wallet addresses or HD paths](https://github.com/joinmarket-webui/jam/pull/1440) | Adds a Sign Message tool allowing users to sign arbitrary messages using wallet addresses or BIP-84 HD derivation paths. |
| [**#1444** — fix(ui): improve balance readability in tooltips](https://github.com/joinmarket-webui/jam/pull/1444) | Resolves digit de-emphasis and color contrast issues for satoshi balance displays inside UI tooltips. |

### 3. [joinmarket-ng/joinmarket-ng](https://github.com/joinmarket-ng/joinmarket-ng)

| PR | What it solves |
|---|---|
| [**#547** — feat(jmwalletd): add backend field to getinfo endpoint response](https://github.com/joinmarket-ng/joinmarket-ng/pull/547) | Adds daemon metadata and active backend identifiers to the `getinfo` API endpoint response. |
| [**#571** — fix(jmwalletd): apply LOGGING__LEVEL configuration to log sinks](https://github.com/joinmarket-ng/joinmarket-ng/pull/571) | Applies environment `LOGGING__LEVEL` configuration dynamically to stdout and file logging sinks in `jmwalletd`. |
| [**#584** — fix(jmwalletd): record direct sends in history file at broadcast time](https://github.com/joinmarket-ng/joinmarket-ng/pull/584) | Implements three-phase transaction lifecycle persistence (Prepare, Broadcast, Finalize) for zero-data-loss history recording. |
| [**#585** — fix(jmwallet): add neutral amount field and nullify coinjoin fields on non-collaborative transactions](https://github.com/joinmarket-ng/joinmarket-ng/pull/585) | Standardizes history outputs with a neutral `amount` field while nullifying `cj_amount` and `source_mixdepth` for direct transactions. |
| [**#590** — fix(taker): map max_sweep_fee_change setting](https://github.com/joinmarket-ng/joinmarket-ng/pull/590) | Maps `max_sweep_fee_change` into `TakerConfig` and enforces relative fee tolerance runtime guards during CoinJoin execution. |

### 4. [joinmarket-webui/joinmarket-ng-api-ts](https://github.com/joinmarket-webui/joinmarket-ng-api-ts)

| Commits | What it does |
|---|---|
| [**feat: migrate to JoinMarket-NG OpenAPI spec and regenerate client**](https://github.com/joinmarket-webui/joinmarket-ng-api-ts/commit/45abe475586a3501d31a2ba59a63efbaa44706ab) | Updates API spec schema definitions and regenerated TypeScript client bindings for `jmwalletd`. |
| [**feat: sync openapi spec with joinmarket-ng backend and regenerate client**](https://github.com/joinmarket-webui/joinmarket-ng-api-ts/commit/73bcd7c22a675bb446f28d0f4e838be2cca3a75b) | Syncs OpenAPI schemas with new backend endpoint parameters. |

---

## What Remains

While significant progress was achieved this summer, open source is an ongoing effort. Future areas of work include:

- E2E automated playwright tests to ensure the stability of Jam.


---

## What I Learned

This summer was a transformative experience for me as an open-source developer:

1. **Full-Stack Bitcoin Engineering**: I gained deep intuition into Bitcoin privacy protocols, CoinJoin mechanics, UTXO mixdepth management, fee tolerance limits, and fidelity bond incentives.
2. **Systemic API & Schema Design**: I learned the immense value of contract-first API development using OpenAPI code generation, bridging Python backends with React UIs seamlessly.

---

## What Happens Next

I wish to continue to be an active contributor across JAM projects. I look forward to helping review incoming pull requests, assisting new contributors, and continuing to build powerful and privacy oriented Bitcoin interfaces.

---

## Beyond the Code: Mentorship

A big thanks to [TBK](https://github.com/theborakompanioni) for being such a great mentor throughout the program. He was always available when I needed help, whether it was around technical decisions, Bitcoin concepts, trade-offs, or review feedback. He was also quick with PR reviews and always willing to discuss even small design decisions and edge cases. What I valued most was the freedom and trust he gave me to express my ideas, make decisions, and figure things out on my own rather than simply telling me what to do. That autonomy helped me become more confident in taking ownership of my work. I’m genuinely grateful and thankful for his patience, trust, and mentorship throughout SoB.

---

## Acknowledgement

Thanks to **Adi and the Summer of Bitcoin team** for providing me with this wonderful opportunity, and to the **JAM maintainers and fellow contributors** who reviewed my work, shared feedback, and encouraged me throughout the summer. I’m grateful for all the discussions, support, and learning that made this experience so meaningful.
