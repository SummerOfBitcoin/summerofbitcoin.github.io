---
layout: post
title: "Making Rust a First-Class Target for ContextVM"
date: 2026-08-23
author: Harsh Chandwani
categories: [Nostr, Development, Open-Source, Stories]
---

The first bug I fixed on `rs-sdk` was three lines long. A constant called `DEFAULT_LRU_SIZE` sat in the constants file, correctly named, correctly sized, and wired to absolutely nothing. Relays in Nostr are allowed to deliver the same event more than once. That's not a failure mode, that's the design. So without a seen-set, one gift-wrapped request could be decrypted and dispatched two or three times. The TypeScript SDK tracked outer event IDs. The Rust one meant to, and didn't.

That gap is the whole shape of my summer, scaled up. ContextVM has a mature TypeScript SDK and a spec that keeps moving. My Summer of Bitcoin project was to bring the Rust implementation up to parity with both: not as a port that mostly works, but as an implementation you could point a TypeScript client at and have it not notice the difference.

Over about four and a half months that came to **53 merged pull requests, roughly 38,000 lines added** and 3,000 removed, across seven ContextVM Enhancement Proposals. Here's what actually happened.

## What ContextVM is, briefly

The Model Context Protocol (MCP) is how AI models talk to tools and data sources. It's normally spoken over stdio or HTTP, both of which assume you know where the server is and that someone is hosting it for you.

ContextVM moves MCP onto Nostr. Servers announce themselves as Nostr events, clients discover them by public key, and JSON-RPC messages travel as relay events, optionally gift-wrapped so the relay can't read them. The addressing is a pubkey instead of a URL, so there's no domain to seize and no host to ask permission from. That's the pitch, and it's a good one, but it means every assumption a normal transport makes has to be re-derived. Relays don't guarantee ordering. Relays duplicate. Events have a size cap. There's no connection to hang state off.

Most of my summer was spent in the space between "MCP assumes X" and "Nostr does not provide X."

## Starting where the tests were missing

I didn't start by writing features. I started by writing tests for code that already existed, because I couldn't tell what was correct.

The first month was conformance tests: [core types](https://github.com/ContextVM/rs-sdk/pull/7), the [initialization flow wire format](https://github.com/ContextVM/rs-sdk/pull/12), then [`tools/list`, `tools/call`, and server announcements](https://github.com/ContextVM/rs-sdk/pull/16) pinned byte-for-byte against what the TypeScript SDK emits. These are boring PRs. They're also the reason every later PR could move fast. Once the wire format is pinned by a test, a refactor either preserves it or turns the suite red.

Then came the piece that unlocked everything else. Both transports took a concrete `Arc<RelayPool>`, which meant every integration test needed a real relay and a real socket. I [extracted a `RelayPoolTrait` and built a `MockRelayPool`](https://github.com/ContextVM/rs-sdk/pull/30), an in-memory relay modeled on the TS SDK's `MockRelayHub`. No sockets, no network, fully deterministic.

That mock is the single highest-leverage thing I built all summer. Every end-to-end test I wrote afterwards, and there are a lot of them, runs against it. It also had its own bug: [live broadcasts ignored subscription filters](https://github.com/ContextVM/rs-sdk/pull/90), so tests were passing because every subscriber saw every event, which is not what a relay does.

## Two bugs that taught me what "server" means here

**Only one client could connect.** The rmcp worker had a field called `active_client_pubkey` and a guard that silently dropped messages from anyone else. One client per server instance. The obvious fix looked like a worker pool, but the TS SDK doesn't do that; it multiplexes every client through one transport. The real problem was subtler: two clients both starting their JSON-RPC request IDs at `1` would collide. So I [removed the guard and gave every inbound request an internal correlation ID](https://github.com/ContextVM/rs-sdk/pull/60) from a monotonic counter, restoring the client's original ID on the way out. Invisible to clients, and the single-peer barrier disappeared. The unbounded session `HashMap` became an LRU capped at 1000, matching TS, with eviction that recreates rather than drops sessions holding in-flight requests.

**One stranger's event killed the server for everyone.** This one is my favourite finding of the summer. rmcp's pre-service handshake accepts exactly one thing as its first message: an `initialize` request. Anything else returns `ExpectedInitializeRequest`, which drops the transport, cancels the worker through its drop guard, and closes everything. Nothing brings it back.

The default allowlist is empty, so any stranger's event gets parsed and dispatched. But you don't even need an attacker. Nostr does not order delivery, so an honest client whose `notifications/initialized` arrives before its own `initialize` kills the server it's trying to talk to.

The fix was to stop letting inbound traffic drive the handshake at all: [the worker satisfies it itself at startup](https://github.com/ContextVM/rs-sdk/pull/110), sending a synthetic `initialize` + `initialized` pair into the handler channel before the event loop drains anything. That channel is single-producer and FIFO, so it's always ahead of every inbound event for every peer and every arrival order. Race-free by construction rather than by timing luck. Eight new tests, seven of them red before the fix, and three deliberate mutations (drop the startup drive, swap the two sends, move it after the loop starts) all die.

## CEP-22: sending things bigger than an event

Nostr events have a size limit. MCP responses do not care about your size limit.

CEP-22 defines oversized transfer: split the payload into chunks, frame them, reassemble on the other side. I built it across [four](https://github.com/ContextVM/rs-sdk/pull/88) [PRs](https://github.com/ContextVM/rs-sdk/pull/89) [in](https://github.com/ContextVM/rs-sdk/pull/91) [sequence](https://github.com/ContextVM/rs-sdk/pull/92), deliberately starting with a transport-agnostic framing engine that was pure dead code until the wiring landed: `OversizedFrame` (Start/Accept/Chunk/End/Abort), a five-class error taxonomy, and a stateful receiver with admission control, out-of-order buffering, and triple validation at the end (byte length, then SHA-256, then JSON-RPC parse).

Two details I'm glad I sweated:

- The byte split is UTF-8 char-aware, so a chunk boundary never lands mid-codepoint.
- Gap arithmetic uses `i128` rather than the obvious `i64`, so a crafted `u64::MAX` progress value can't overflow or wrap. TypeScript sidesteps this for free via JS floats; Rust does not, and a hardening that costs nothing is worth taking.

The hardest part wasn't framing. It was timeouts. A chunked response longer than the idle timeout would time out mid-transfer even while making perfect steady progress. So inbound frames are forwarded as stripped progress notifications to reset the requester's idle timer, and the numeric `progressToken` type has to be preserved exactly through the restore path, because rmcp's watcher map is type-exact and `Number(5)` is not `String("5")`. Add a per-transfer watchdog on top, flip the feature on by default to match TS, and CEP-22 was done at 562 tests passing.

## CEP-41: streaming, and a bug that was already there

CEP-22 chunks a response you already have. CEP-41 is the other thing: a tool that produces output over time and wants to push it as it goes.

I built it as a [pure engine](https://github.com/ContextVM/rs-sdk/pull/97) and then [wired both transports](https://github.com/ContextVM/rs-sdk/pull/98). The server creates a writer on any `tools/call` carrying a `progressToken` and captures a route snapshot at creation, so the deferred final response can still be delivered after the route store has swept the entry. Response deferral sits at the top of `send_response`: a started writer stashes the response and returns; an unstarted one drops through and sends normally.

Wiring it surfaced a bug that predated the feature. The oversized reassembly path dispatched `tools/call` directly, bypassing writer creation entirely, so an oversized request carrying a `progressToken` never got a writer and never streamed. Two features that each worked, silently broken at their intersection. Fixed by creating the writer on the reassembled end frame, and there's now an e2e test that composes CEP-22 and CEP-41 specifically to keep that honest. 737 tests passing.

## CEP-8: payments, and a race TypeScript gets for free

The last stretch has been CEP-8: capability pricing and payment over ContextVM. This is where the port stopped being a port.

I built it in layers: [primitives](https://github.com/ContextVM/rs-sdk/pull/102) (tags, wire types, traits), [canonical invocation identity](https://github.com/ContextVM/rs-sdk/pull/103), an [authorization store](https://github.com/ContextVM/rs-sdk/pull/104), a [general-purpose inbound middleware seam](https://github.com/ContextVM/rs-sdk/pull/101), then negotiation on the [server](https://github.com/ContextVM/rs-sdk/pull/107) and [client](https://github.com/ContextVM/rs-sdk/pull/108) transports, and finally the two lifecycles where money actually moves: [transparent gating](https://github.com/ContextVM/rs-sdk/pull/111) and [explicit gating](https://github.com/ContextVM/rs-sdk/pull/112).

The most interesting thing I've written all summer is one method: `claim_or_set_pending`.

TypeScript runs `claim()` and then `trySetPending()` back to back and gets atomicity for free, because JS is run-to-completion, and nothing can interleave between two synchronous calls. Rust cannot make that assumption. The payment verification runs on a detached task on another OS thread, and the literal two-call port has a real window where a settlement pops the pending state *between* the two calls. The server then mints a second invoice while an already-paid grant sits unclaimed. The user pays twice.

I didn't want to argue about whether the race was real, so I measured it: a two-thread barrier probe trips the two-call port roughly **ten times per 100,000 contended rounds**, and the composed single-critical-section op never trips. Every observable outcome of the composed op is one TypeScript also produces. It just can't be reached by an interleaving that TS's runtime forbids and Rust's doesn't.

That's the lesson I'd hand to anyone porting between these two languages: the reference implementation encodes its runtime's guarantees in its *structure*, not its comments. Copying the structure into a language with different guarantees copies the shape and drops the safety.

Two smaller findings from the same stretch got filed upstream against the TypeScript SDK: it sends payment errors correlated on the Nostr event ID rather than the request's own JSON-RPC ID, which breaks any client correlating errors the normal way; and a `ttl: 0` payment option mints a grant that expires at birth while its invoice stays payable.

## The rest of it

Not everything was a protocol feature, and the unglamorous work mattered:

- [CI for fmt, clippy, and docs](https://github.com/ContextVM/rs-sdk/pull/22), then [MSRV and a feature matrix](https://github.com/ContextVM/rs-sdk/pull/75), because "works on my machine with all features on" is not a claim about a library.
- A [`missing_docs` lint with every rustdoc gap closed](https://github.com/ContextVM/rs-sdk/pull/73), so the gap can't reopen.
- [CEP-35 discovery tags](https://github.com/ContextVM/rs-sdk/pull/51) with server and client [tag emission and capability learning](https://github.com/ContextVM/rs-sdk/pull/57).
- [CEP-6 announcements](https://github.com/ContextVM/rs-sdk/pull/78): auto-publishing on start, [relay lists and profile metadata](https://github.com/ContextVM/rs-sdk/pull/79).
- [CEP-17 multi-stage relay resolution](https://github.com/ContextVM/rs-sdk/pull/83), finding which relays a server is actually reachable on.
- Two releases shipped: [0.1.0](https://github.com/ContextVM/rs-sdk/pull/68) and [0.2.0](https://github.com/ContextVM/rs-sdk/pull/95), plus the [rmcp 0.16 to 1.7 upgrade](https://github.com/ContextVM/rs-sdk/pull/86).
- A [pending-request leak](https://github.com/ContextVM/rs-sdk/pull/61) fixed with a TTL sweep, [event loop tasks that didn't stop on `close()`](https://github.com/ContextVM/rs-sdk/pull/63), and an [allowlist that returned nothing instead of `-32000 Unauthorized`](https://github.com/ContextVM/rs-sdk/pull/53).

## On bindings

One of the project's stated outcomes was a note on future FFI. It ended up less of a note and more of a constraint I carried the whole way: the FFI surface is mirrored as features land, not bolted on afterwards. Payment errors got a dedicated `CVM_PAYMENT = 8` code kept in lockstep across the Rust enum, the C header, and the C test. CEP-41's `supports_open_stream` was mirrored out to Python, Swift, and Kotlin consumers during review of that PR rather than in a follow-up.

The argument for a Rust implementation was never that Rust is nicer than TypeScript. It's that one correct Rust core can become the Python SDK, the Swift SDK, and the Kotlin SDK without anyone re-deriving the framing engine or the payment race three more times. That only holds if the FFI layer never falls behind, so I treated "did the binding surface move?" as part of the definition of done.

## What I'd tell the version of me from April

**Write the mock first.** I spent weeks writing tests that needed a live relay before building `MockRelayPool`. Every one of those weeks would have been faster on the other side of that refactor.

**Read the reference implementation for its reasons, not its lines.** The instinct when porting is to make the diff look like the original. The times I learned the most were the times I couldn't: the `claim_or_set_pending` race, the `i128` gap arithmetic, the synthetic handshake. Each of those is a place where the TS code is correct *for TypeScript* and copying it faithfully into Rust would have shipped a bug.

**Bugs live at intersections.** The CEP-22 and CEP-41 bug, the deferred-response path missing its discovery tags, the oversized deferred publish that never fragments. None of these are in a feature. They're all in the seam between two features that each pass their own tests. I now write a composition test whenever two features can be on at once.

**Write the PR description for the reviewer who will ask.** My later PRs have a "things you'll probably ask about" section, and it's the single change that most improved review turnaround. Anticipating the objection is cheaper than defending against it in a comment thread three days later.

## What's next

Two payment PRs are open and in review: the transparent and explicit gating middlewares. Client-side registration and the retry loop are stacked behind them. Beyond that, the follow-ups I filed off the handshake fix point at a real architectural question: one rmcp service serving every client is the root of three separate cross-client issues, and one service per client pubkey closes all of them at once. That's a worker rewrite, not a patch, and it's the thing I most want to see happen next.

Rust isn't quite a first-class ContextVM target yet, but it's close enough that the remaining gaps are ones I can name, and that feels like the right place to be at the end of a summer.

---

Thanks to the ContextVM team for review that was consistently sharper than my PRs deserved, and to Summer of Bitcoin for the space to work on something this deep. The SDK is at [github.com/ContextVM/rs-sdk](https://github.com/ContextVM/rs-sdk), the spec at [docs.contextvm.org](https://docs.contextvm.org), and issues are open if you want to help.
