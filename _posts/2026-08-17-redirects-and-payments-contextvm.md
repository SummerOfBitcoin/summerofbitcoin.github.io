---
layout: post
title: "Redirects, Payments, and Shipping a Full Summer at ContextVM"
date: 2026-08-17
author: Abhay Gupta
categories: [Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-08-17-redirects-and-payments-contextvm.jpg
---

This is my final project report for the Summer of Bitcoin 2026 program. Over the past twelve weeks, I worked on building core infrastructure for ContextVM, an ecosystem centered on decentralized deployment and discovery of Model Context Protocol (MCP) servers over the Nostr network.

This post covers what I set out to do, what I actually built, the major technical decisions I had to make, what shipped, what remains, and what I learned along the way.

## What I Set Out to Do

My original project proposal focused on implementing foundational standards and tooling for ContextVM. The scope covered three main areas:

*   **Server distribution:** Building a packaging and deployment pipeline so developers could bundle their MCP servers into distributable archives and run them over Nostr.
*   **SDK optimization:** Reducing the installation footprint of the official TypeScript SDK by cleaning up heavyweight dependencies.
*   **Protocol standards:** Contributing to ContextVM Enhancement Proposals (CEPs) for interoperability, server discovery, and payments.

By the midterm checkpoint, I had already shipped the SDK optimization and website refactoring work. The second half of the program was entirely dedicated to implementing CEP-47 Server Redirect support in the SDK.

## What I Built

Here is a breakdown of every major piece of work I delivered across the full twelve weeks.

### First Half (Weeks 1 to 6)

**SDK Dependency Optimization** ([ContextVM/mcp-sdk#3](https://github.com/ContextVM/mcp-sdk/pull/3) - Merged)
I conducted a dependency audit of the official TypeScript SDK. By moving heavy validation libraries like Zod and Ajv into optional peer dependencies, I reduced the default `node_modules` install size by about 3.5 MB without breaking any existing implementations.

**Website UX Overhaul** ([ContextVM/contextvm-site#30](https://github.com/ContextVM/contextvm-site/pull/30), [#28](https://github.com/ContextVM/contextvm-site/pull/28) - Merged)
I redesigned the main site header, added dynamic SEO and Open Graph improvements, and broke down a monolithic 800-line Svelte landing page into clean, reusable components.

**MCPB Server Bundling** ([ContextVM/cvmi#4](https://github.com/ContextVM/cvmi/pull/4) - Approved, pending merge)
I built the `cvmi pack` command that packages MCP server projects into compressed `.mcpb` zip archives, and extended the `cvmi serve` command to extract and run those bundles over Nostr. The implementation validates a custom `_meta.com.contextvm` manifest namespace using Zod schemas. This PR has been reviewed and approved by my mentor and is awaiting merge.

**CEP-15 Common Tool Schemas** ([CEP-15 Specification](https://github.com/ContextVM/contextvm-docs/blob/master/src/content/docs/reference/ceps/cep-15.md))
I helped draft and formalize the specification for Common Tool Schemas, which uses deterministic hashing (RFC 8785) and CEP-6 tags so clients can discover and switch between standardized service providers.

### Second Half (Weeks 7 to 12)

**CEP-47 Server Redirect Middleware** ([ContextVM/sdk#78](https://github.com/ContextVM/sdk/pull/78) - Open, under review)
This was the main deliverable for the second half of the program. I implemented full end-to-end server redirect support for `@contextvm/sdk`, covering both the server-side and client-side middleware.

## Major Technical Decisions

### Designing the Redirect Middleware

The biggest design decision in CEP-47 was how to structure the redirect flow without breaking the SDK's existing stateless initialization lifecycle.

**The problem:** When a client connects to Server A and Server A decides this request should be handled by Server B, the server needs to reject the request with a redirect signal, and the client needs to transparently reconnect to Server B without the caller knowing anything happened.

**The approach I chose:** Following my mentor's guidance, I went with a callback-driven middleware pattern:

*   **Server-side** (`withServerRedirect`): A middleware that accepts a `resolveRedirect` callback. It evaluates the incoming JSON-RPC request and, if a redirect is needed, emits the standard `-32044` MCP error payload containing the target server's pubkey and relay list. This keeps the redirect logic completely decoupled from state configuration.

*   **Client-side** (`withClientRedirect`): Intercepts incoming messages looking for the `-32044` error code. When it detects one, it transparently spins up a new `NostrClientTransport` to the target server, routes all subsequent messages through the new transport, and replaces the old connection. The caller never sees the redirect happen. I also added cycle/loop detection with a configurable `maxRedirects` option (defaulting to 5 hops) so that circular redirects (A to B to A) throw a proper `McpError` instead of hanging forever.

### Why Callback-Driven Instead of Config-Driven

The alternative was to bake the redirect rules directly into the server's state configuration. My mentor specifically steered me away from this because it would have coupled the redirect logic to the initialization lifecycle, making the middleware harder to compose and test independently. The callback approach keeps things stateless and lets the server operator define any redirect logic they want without touching the SDK internals.

## What Shipped

| Contribution | Repository | Status |
|---|---|---|
| SDK dependency optimization | [mcp-sdk#3](https://github.com/ContextVM/mcp-sdk/pull/3) | Merged |
| Website header redesign and SEO | [contextvm-site#30](https://github.com/ContextVM/contextvm-site/pull/30) | Merged |
| Website landing page componentization | [contextvm-site#28](https://github.com/ContextVM/contextvm-site/pull/28) | Merged |
| MCPB server bundling CLI | [cvmi#4](https://github.com/ContextVM/cvmi/pull/4) | Approved, pending merge |
| CEP-47 server redirect middleware | [sdk#78](https://github.com/ContextVM/sdk/pull/78) | Under review |
| CEP-15 Common Tool Schemas spec | [CEP-15](https://github.com/ContextVM/contextvm-docs/blob/master/src/content/docs/reference/ceps/cep-15.md) | Draft |

## What Remains

*   **sdk#78 (CEP-47 redirect)** is currently open and under review. The code is complete with full E2E test coverage, but it still needs the final sign-off from the maintainers.
*   **cvmi#4 (MCPB bundling)** has been approved and is waiting to be merged into the main branch.
*   The original project proposal also mentioned CEP-8 Payment Gateway integration. During the second half, the scope shifted to CEP-47 Server Redirects based on discussions with my mentor, as redirect support was a prerequisite for the broader payment flow to work correctly across the network.

## What I Learned

**Middleware composition matters more than you think.** Before this project, I had a surface-level understanding of middleware patterns. Building `withServerRedirect` and `withClientRedirect` taught me why stateless, callback-driven middleware is so much easier to test and maintain than tightly coupled alternatives. Writing the test suite for chained redirects (Client to Server A to Server B to Server C) forced me to think very carefully about transport lifecycle management.

**Bug fixes teach you the most about a codebase.** While building the redirect middleware, I ran into two bugs in the existing SDK that I had to fix first. One was in `ApplesauceRelayPool` where the message handler was expecting typed wrapper objects, but `applesauce-relay` was actually emitting raw events. The other was a double-prepending issue in `mock-relay-server` that caused test relay timeouts. Debugging these gave me a much deeper understanding of how the Nostr transport layer works under the hood than I would have gotten from just writing new code.

**Scope changes are normal.** The original plan for the second half was to implement CEP-8 payments. After discussing the architecture with my mentor, we realized that server redirects (CEP-47) needed to land first because the payment flow depends on being able to redirect clients to the correct payment-gated server. Adjusting the scope mid-program felt uncomfortable at first, but it was the right engineering call.

## What Happens Next

Even though the program is wrapping up, I plan to stay involved with ContextVM. The immediate next steps are:

*   Getting sdk#78 through review and merged.
*   Getting cvmi#4 merged once the maintainers are ready.
*   Circling back to CEP-8 payment gateway integration, now that the redirect infrastructure is in place to support it.

---

## Links

*   **Website:** [abhayakg.me](https://abhayakg.me)
*   **Email:** [abhayakg123@gmail.com](mailto:abhayakg123@gmail.com)

**All Contributions:**
*   [ContextVM/mcp-sdk#3](https://github.com/ContextVM/mcp-sdk/pull/3) (SDK optimization - Merged)
*   [ContextVM/contextvm-site#30](https://github.com/ContextVM/contextvm-site/pull/30) (Website UX - Merged)
*   [ContextVM/contextvm-site#28](https://github.com/ContextVM/contextvm-site/pull/28) (Website UX - Merged)
*   [ContextVM/cvmi#4](https://github.com/ContextVM/cvmi/pull/4) (MCPB bundling - Approved)
*   [ContextVM/sdk#78](https://github.com/ContextVM/sdk/pull/78) (CEP-47 redirects - Under review)
*   [CEP-15 Specification](https://github.com/ContextVM/contextvm-docs/blob/master/src/content/docs/reference/ceps/cep-15.md) (Common Tool Schemas - Draft)
