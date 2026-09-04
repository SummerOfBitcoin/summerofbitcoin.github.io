---
layout: post
title: "From Protocol to Product: Routing Agentic Computation Over Nostr"
date: 2026-08-18
author: "Khushvendra Singh"
categories: [Nostr, Lightning-Network, Development, Open-Source, AI, Stories]
image: assets/images/blog_content/nostr_mcp.png
---

At the midpoint of Summer of Bitcoin, I ended my [first report](https://blog.summerofbitcoin.org/routing-agentic-computation-over-nostr/) with a concrete goal: a user should be able to ask an LLM to call a remote tool, receive a Lightning invoice when that tool requires payment, pay it, and continue the same workflow.

That end-to-end path is now merged.

The sentence is simple, but the system behind it is not. A single call crosses an LLM, an agent loop, the Model Context Protocol (MCP), a Nostr transport, a payment protocol, and a user interface. If any layer drops an error, retries the wrong request, or waits forever, the whole experience breaks.

My Summer of Bitcoin project became the work of making that entire path reliable.

## What I set out to do

[ContextVM](https://github.com/ContextVM) connects MCP to Nostr. A service can advertise tools over Nostr, a client can discover and call them through relays, and paid capabilities can settle over the Lightning Network. This removes the need for every tool to live behind a fixed, centrally hosted API.

I set out to help make this model practical for real applications. The work had three main goals:

1. Support long-running, incremental results instead of assuming every call ends with one response.
2. Build a maintainable transport and a browser client that an LLM can use safely.
3. Carry payment requirements from a remote server to the user without losing context, then retry the exact authorized call.

The project spread across the ContextVM specification, TypeScript SDK, website, inference service, CLI, and documentation repositories. No single pull request tells the full story; the important result is how those pieces now work together.

## Building the path, layer by layer

### Open-ended streams for work that does not finish at once

Agentic computation often produces partial results: generated tokens, progress updates, intermediate tool output, or a stream whose length is not known in advance. A normal request-response exchange is too restrictive for that.

I helped refine and harden [CEP-41, the open-ended-stream specification](https://github.com/ContextVM/contextvm-docs/pull/40), and its [TypeScript transport implementation](https://github.com/ContextVM/sdk/pull/71) through protocol and code review. The review surfaced the less visible parts of streaming: bounded buffers, stale chunks, aborts, pings, shutdown, and cleanup. These edge cases matter because a stream that never releases its state is not merely slow; it can leave future requests stuck behind it.

Both the specification and SDK support were merged. CEP-41 later became important again in the command-line work, where streamed Muxll output had to be consumed and rendered as it arrived.

### A transport organized around real responsibilities

The SDK transport had grown into two files of more than a thousand lines each. My first instinct was to make them smaller. Review changed the question from "What code can move?" to "What complete responsibility belongs somewhere else?"

That led to the [transport-layer refactor](https://github.com/ContextVM/sdk/pull/72). I separated inbound event coordination, outbound request sending, response routing, capability negotiation, oversized-message handling, and open-stream lifecycle management into focused modules. The main client and server transports remained the public facades, so the external API did not change.

The distinction was important. Extracting a few helpers can reduce a file's line count while making the execution path harder to follow. Extracting a complete protocol workflow gives state and behavior a clear owner. The public API stayed unchanged, and the transport test suite stayed green as the refactor grew through review.

### From a chat demo to a bounded agent loop

I first built a configurable [Bring Your Own Token chat demo](https://github.com/gzuuus/sv5-chat-demo/pull/1). I then carried that work into ContextVM's [production web chat](https://github.com/ContextVM/contextvm-site/pull/29), with provider selection, streaming responses, model fallback, cancellation, and browser-side conversation persistence.

The next step was to let the model use remote MCP tools. The [agentic function-calling integration](https://github.com/ContextVM/contextvm-site/pull/31) introduced two pieces that became central:

- An `AgentOrchestrator` owns the multi-turn loop, limits it to five rounds, propagates cancellation, and pauses for approval before sensitive tools run.
- A multi-server `ToolRegistry` maps model-facing tool names back to the correct Nostr server and resolves name collisions using the server's public key.

This kept orchestration out of the Svelte components and made it testable on its own. It also made safety part of the control flow rather than a UI afterthought: loops are bounded, approvals expire, and cancellation reaches pending model and tool calls.

I also completed the [documentation restructuring](https://github.com/ContextVM/contextvm-docs/pull/45) tracked in [issue #43](https://github.com/ContextVM/contextvm-docs/issues/43). The new structure separates reference material, how-to guides, and tutorials, improves protocol discoverability, and adds 12 Rust SDK pages. The complete 60-page site passed a clean Astro build.

## Closing the loop with explicit payments

At midterm, the remaining core task was connecting the SDK's payment lifecycle to the web client.

The [CEP-8 explicit-gating implementation](https://github.com/ContextVM/sdk/pull/75) made payment part of the request-response protocol. A server can reject a priced call with `-32042 Payment Required`, report an in-progress payment with `-32043 Payment Pending`, and execute only after authorization. The client can handle the payment and retry the original request.

The hardest requirement was idempotency. A network retry must not charge for one invocation and execute another, or execute the paid call twice. The SDK therefore creates a canonical invocation identity from the method, parameters, client public key, and JSON-RPC request ID. It canonicalizes that data with RFC 8785 JCS and hashes it with SHA-256. A bounded authorization store then uses that identity to match payment and execution.

After midterm, I completed the [web-client integration](https://github.com/ContextVM/contextvm-site/pull/33). Payment errors now travel losslessly through the transport and agent loop as structured `payment_required` results. The chat can show invoices, QR codes, and multiple payment options without hardcoding one payment method. The server page can negotiate either the new `explicit_gating` mode or the older `transparent` mode, so existing servers and clients continue to work.

The final path is now:

> Prompt → LLM tool call → MCP over Nostr → payment requirement → Lightning invoice → authorized retry → streamed result

This integration is the best single representation of my project because it joins protocol design, transport behavior, payment safety, agent orchestration, and user experience in one merged workflow.

## Taking the design into billing and CLI workflows

Once the core path worked, I used it to harden the services around it.

[Muxll](https://github.com/ContextVM/muxll) provides LLM inference through ContextVM. In [Muxll PR #3](https://github.com/ContextVM/muxll/pull/3), I aligned its prepaid billing with CEP-8. Users can top up a balance through a Lightning invoice, keep the same prepaid identity across CLI runs, inspect their balance, and receive a clear rejection when funds are insufficient. Funded calls are allowed through the CEP-8 policy layer and actual usage is deducted afterward. A live smoke test covered invoice creation, payment, ledger credit, chat execution, and usage deduction.

Money exposed another class of correctness problems. In [Muxll PR #5](https://github.com/ContextVM/muxll/pull/5), I replaced an operator-managed exchange rate with live BTC/USD data from Kraken, Coinbase, and Binance. The rate is cached for one hour, individual provider failures are tolerated, and top-ups fail closed if every provider is unavailable. I also moved ledger history from floating-point USD values to integer nano-USD, including a transactional migration for existing data. Public APIs still expose familiar USD values; the exact accounting stays internal.

I also hardened [ContextVM's CLI, CVMI](https://github.com/ContextVM/cvmi/pull/6). The CLI now consumes CEP-41 streams, renders Muxll output incrementally, applies bounded request timeouts, falls back when a server does not advertise encryption and encryption is optional, forwards output flags correctly, returns nonzero exit codes for MCP errors, and makes explicit gating opt-in. Its local validation covered 416 tests, type-checking, builds, formatting, and packaged end-to-end calls against a local Nostr relay.

## The design decisions that mattered most

Several principles kept appearing across repositories.

**Extract around ownership, not file size.** A module should own a protocol or lifecycle concern from start to finish. This made the transport easier to change without scattering shared state across arbitrary helpers.

**Preserve errors as data.** Payment requirements are not generic failures. If one layer converts a structured CEP-8 response into a string, the invoice, retry state, and payment options disappear. Lossless propagation made the web integration possible.

**Bound everything that waits.** Agent rounds, approvals, payment retries, transport requests, and stream cleanup all need limits or cancellation. Distributed systems rarely fail with a clean exception; they often stop making progress.

**Treat money as a state machine.** Canonical request identities prevent unsafe retries. Integer accounting prevents floating-point drift. Transactional migration preserves old balances. Live rates fail closed rather than silently using stale data.

**Negotiate new behavior without breaking old clients.** CEP-8 explicit gating is opt-in, the transparent mode still works, and the transport refactor kept its public API. Compatibility made it possible to ship the work in stages.

## What shipped, and what remains

At the time of writing, the agreed core deliverables are complete and merged. The later billing and CLI extensions are implemented in open pull requests; review, integration, and a few CI follow-ups remain.

| Deliverable | Status |
| --- | --- |
| [CEP-41 specification](https://github.com/ContextVM/contextvm-docs/pull/40) and [SDK transport support](https://github.com/ContextVM/sdk/pull/71) | Merged |
| [Transport decomposition and hardening](https://github.com/ContextVM/sdk/pull/72) | Merged |
| [Production LLM chat](https://github.com/ContextVM/contextvm-site/pull/29) and [MCP agent loop](https://github.com/ContextVM/contextvm-site/pull/31) | Merged |
| [CEP-8 SDK lifecycle](https://github.com/ContextVM/sdk/pull/75) | Merged |
| [End-to-end web payment integration](https://github.com/ContextVM/contextvm-site/pull/33) | Merged |
| [Documentation restructuring](https://github.com/ContextVM/contextvm-docs/pull/45) | Merged |
| [Muxll prepaid billing](https://github.com/ContextVM/muxll/pull/3), [integer ledger and live rates](https://github.com/ContextVM/muxll/pull/5), and [dependency upgrade](https://github.com/ContextVM/muxll/pull/6) | Open; review and integration pending |
| [CVMI streaming, timeout, error, and payment hardening](https://github.com/ContextVM/cvmi/pull/6) | Open; review pending |

The [original web-client tracking issue](https://github.com/ContextVM/contextvm-site/issues/32) is still open, but its requested explicit-gating behavior was implemented by the merged web PR. CVMI still has review and CI feedback to resolve, and the open Muxll branches may need review-driven cleanup or rebasing before merge. No originally agreed core deliverable was dropped.

## What I learned

My biggest technical lesson is that end-to-end correctness is different from local correctness. The SDK can return the right error, but the feature still fails if the agent loop flattens it. A server can verify a payment, but the system is still unsafe if a retry points to a different invocation. Each boundary has to preserve the meaning established by the previous one.

I also learned that payment changes ordinary networking decisions into financial decisions. A duplicate retry is no longer harmless. Rounding is no longer cosmetic. A stale exchange rate is no longer only a data-quality issue. This pushed me to think in terms of identities, transitions, atomic operations, and failure policies instead of isolated functions.

Review was just as important as implementation. During the transport refactor, my mentor pushed me away from cosmetic extraction and toward protocol and lifecycle boundaries. That feedback changed the final architecture and gave me a rule I will keep using: a useful abstraction makes responsibility clearer, not merely the file shorter.

Finally, I learned the value of testing beyond unit boundaries. The most convincing validation was not a green test count; it was watching a real Lightning invoice get paid, seeing the ledger credit arrive, running the chat request, and confirming that actual usage was deducted. Tests make changes safe. End-to-end checks show whether the system is useful.

## What happens next

My immediate next step is to address the remaining review and CI feedback, then work with the maintainers to integrate the open Muxll and CVMI pull requests. After that, I plan to keep testing these flows against real relays, wallets, models, and failure conditions, while improving the documentation for people building on ContextVM.

I also plan to continue contributing after Summer of Bitcoin. My collaboration with ContextVM began before the official program results, and the project never felt like an isolated internship exercise. I was trusted with production-facing problems, received careful architectural feedback, and saw my work become part of a live open-source system.

I am grateful to [gzuuus](https://github.com/gzuuus), the ContextVM maintainer, and Summer of Bitcoin for that trust. I started the summer trying to make agentic computation over Nostr possible. I am ending it with the full path merged, the next layer built, and a much clearer understanding of what it takes to turn a protocol idea into a dependable product.


