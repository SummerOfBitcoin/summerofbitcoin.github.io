---
layout: post
title: "Dollar Stability, Backed by Bitcoin: Detangling and Improving the Stable Channels LSP"
date: 2026-08-22
author: Biresh Biswas
categories: [Lightning-Network, Wallets, Security, Development, Open-Source]
image: ../assets/images/blog_content/2026-08-22-stable-channels-lsp.png
---

Stable Channels has a simple goal: **dollar stability, backed by Bitcoin**. It lets two peers use a Lightning channel to maintain a USD-denominated balance without a stablecoin, fiat account, or custodian. As the Bitcoin price changes, Lightning payments adjust the channel allocation so the Stable Receiver stays close to its chosen dollar value.

My Summer of Bitcoin 2026 project focused on the Lightning Service Provider (LSP) that makes this usable. I set out to improve the server's reliability, give operators a practical dashboard, and package the system for Umbrel. By the end of the program, the LSP had been separated from a forked LDK Server, the deployed server had been migrated to the new architecture, the operator interface had grown substantially, and the protocol had a much stronger foundation for safe iteration.

Watch the [project demo video](https://x.com/Biresh_Biswas/status/2091197025367007265?s=20) for a walkthrough of the legacy single-process design, the new two-daemon architecture, and the improved Stable Channels wallet–LSP protocol.

## The starting point

At the beginning of the program, the Stable Channels LSP and its Lightning node lived in one process. Stable-Channels-specific code was embedded inside a fork of LDK Server alongside channel management, payments, chain access, and the stability logic.

This worked as an early implementation, but it tightly coupled the product to the fork. Adopting improvements from upstream LDK Server also meant maintaining and migrating the Stable Channels integration. A problem in the product-specific layer could affect the Lightning node process itself.

The protocol also covered the basic successful path better than its failure paths. For example, some results were inferred from a successful fee payment instead of being explicitly confirmed by the LSP. Messages did not always carry enough context to correlate a result with one exact request, and stability payments needed stronger authentication and replay protection.

## Detangling the architecture

The most important architectural change was splitting the system into two cooperating daemons. I introduced this design in [PR #81](https://github.com/toneloc/stable-channels/pull/81).

- **LDK Server** remains a generic, upstream Lightning node service. It owns the LDK Node, wallet, channels, Lightning payments, and LSPS2 liquidity service.
- **stable-channels-lsp** owns the product-specific state: stable-channel records, price feeds, protocol handling, settlement logic, push notifications, and the operator audit trail.

Wallets and the operator dashboard call the Stable Channels daemon through an authenticated REST API over TLS. When the daemon needs a Lightning operation, it calls vanilla LDK Server through its authenticated gRPC API. The two services use separate API keys and pinned TLS certificates, while the Lightning peer connection carries payments and signed protocol messages between the wallet and the LSP.

This separation created a cleaner boundary: LDK Server handles Lightning infrastructure, and the Stable Channels daemon handles the product. It also allows upstream LDK Server upgrades to be adopted without repeatedly re-embedding Stable Channels code into the server. After validating the new stack, we migrated the deployed LSP to this architecture.

## Building an operator dashboard

Running an LSP requires more than starting a daemon. An operator needs to understand balances, channel liquidity, payments, stable-channel state, and failures without reading raw database rows or searching through unstructured logs.

I expanded the Rust `egui` interface into an operator dashboard. The resulting GUI provides views for node information, on-chain and Lightning balances, channels, payments, forwarded payments, stable channels, and audit records. I also added settlement-aware payment labels, a searchable audit log, and a channel ledger so an operator can follow how a channel's accounting changes over time. The work is represented by the [GUI update](https://github.com/toneloc/stable-channels/pull/188), [searchable audit log](https://github.com/toneloc/stable-channels/pull/155), and [channel ledger](https://github.com/toneloc/stable-channels/pull/200) PRs.

The dashboard works as both a native application and a browser interface. The browser build is also the management interface included in the Umbrel package.

## Improving the protocol

Once the architecture was separated and running, we concentrated on making protocol outcomes explicit, authenticated, correlated, and durable.

### Trades

A wallet now sends a signed trade request that identifies the channel and requested target. The LSP validates the wallet signature, channel binding, quote, freshness, fee, and available capacity. It then returns a signed result correlated to that exact request, including an explicit rejection when the trade cannot be accepted.

The LSP persists its decision before replying, and the wallet applies only an authenticated result that matches its pending request. This matters when a response is lost or either side restarts: the same result can be retried without applying the trade twice. [PR #238](https://github.com/toneloc/stable-channels/pull/238) added durable signed trade rejections and correlated results.

### Synchronization and accounting

Synchronization messages carry monotonic versions so older state cannot overwrite newer state. Both peers derive stable backing from their own trusted local balance and price instead of blindly accepting a peer-supplied accounting value. The protocol also preserves existing stability drift when a trade changes the target, avoiding an accidental reset of value already owed between the peers. These decisions were implemented through [local backing derivation](https://github.com/toneloc/stable-channels/pull/221) and [drift-preserving trades](https://github.com/toneloc/stable-channels/pull/225).

### Stability settlements

Stability payments now carry signed metadata binding the settlement ID, channel, exact amount, direction, target, and validity window. The receiver verifies those fields and records each settlement ID durably, which prevents replay and makes the accounting instruction match the Lightning payment that actually arrived. [PR #233](https://github.com/toneloc/stable-channels/pull/233) implemented authenticated, amount-bound stability settlements.

Across these flows, one design principle became especially important: a successful Lightning payment proves that sats moved, but it does not by itself prove that the related application instruction was accepted. The Stable Channels protocol must authenticate and persist that application-level result separately.

## Umbrel packaging

I packaged the system as a multi-service Umbrel app containing vanilla LDK Server, the Stable Channels LSP daemon, and the browser dashboard. The images support both AMD64 and ARM64, and the complete stack has been tested locally.

The community-store submission is open as a [draft Umbrel PR](https://github.com/getumbrel/umbrel-apps/pull/5948). We are intentionally holding the public launch until upstream LDK Server publishes a stable release, because the package should not expose home-node operators to avoidable pre-stable API changes.

## What shipped

By the end of the program:

- The two-daemon architecture was merged and the deployed LSP was migrated to it.
- Stable Channels could use vanilla upstream LDK Server over gRPC instead of carrying its product logic inside a fork.
- The native and web operator dashboard covered the main operational and accounting views.
- Trade, synchronization, and stability-settlement flows gained signed messages, correlation, replay protection, durable decisions, and stronger local validation.
- The Umbrel package was built and tested locally on the path toward a community-store release.

## What remains

The immediate distribution milestone is to publish the Umbrel app after the stable LDK Server release is available. The protocol will continue to evolve rather than being treated as finished. The next iterations are tracked in the [Stable Channels client and server behavior specification](https://github.com/toneloc/stable-channels/issues/234), including continued protocol review and bringing the same guarantees to every client platform.

## What I learned

This project gave me a much deeper understanding of Bitcoin, Lightning, LDK, and the practical work behind an LSP. The biggest technical lesson was that a protocol is not only its successful flow. It must also define authentication, rejection, correlation, retries, crash recovery, replay handling, and which side is trusted for each piece of state.

I also learned how much open-source development depends on review and clear discussion. I owned the implementation from investigation through testing and iteration, while my mentor, Tony Klausing, reviewed the work, kept priorities clear, and worked through protocol design decisions with me as we improved the protocol.

More broadly, the program strengthened my interest in making Bitcoin technology useful beyond technical early adopters. The tools are still evolving, and the challenge I want to keep working on is turning their advantages into products that ordinary people can understand and use.

## What happens next

Next, we will release the Umbrel package when the upstream dependency is ready, continue the protocol improvements tracked in the specification, and iterate from real deployment experience. The architecture now gives each layer room to improve independently, which is the foundation we needed for that work.

You can follow the project in the [Stable Channels repository](https://github.com/toneloc/stable-channels), read more at [stablechannels.com](https://stablechannels.com), or join the technical discussion through the project's [open issues](https://github.com/toneloc/stable-channels/issues).
