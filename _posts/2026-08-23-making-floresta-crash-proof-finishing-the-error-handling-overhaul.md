---
layout: post
title: "Making Floresta Crash-Proof: Finishing the Error Handling Overhaul"
date: 2026-08-23
author: Ved Patel
categories: [Development, Open-Source, Bitcoin-Core]
image: ../assets/images/blog_content/floresta-error-handling.png
---

In my [midterm blog post](https://summerofbitcoin.github.io/blog/making-floresta-crash-proof-a-systematic-error-handling-overhaul), I described the first half of a systematic error handling overhaul for [Floresta](https://github.com/getfloresta/Floresta), a lightweight Bitcoin full node written in Rust. I had refactored the two largest modules in `floresta-chain` - the on-disk block storage and the chain state manager - replacing crash-prone `unwrap()` calls with typed, recoverable errors and introducing workspace-level lint rules.

That was the foundation. This post covers what happened next: refactoring the remaining 14 modules across five crates, designing an error hierarchy that scales across module boundaries, and flipping the lint configuration from `warn` to `deny` so the compiler itself prevents regressions.

The result: **41 commits, 86 files changed, +5,990 / −1,454 lines**, touching every major module in Floresta, and `cargo clippy` passes with zero errors and zero warnings across the entire workspace. Because this is such a massive structural change to the codebase, I have staged the final consolidated PR on my own fork for now. We will review these changes carefully and gradually before merging them into the organization repository.

## Recap: Where the Midterm Left Off

By the midterm evaluation, I had:

- Designed `FlatChainstoreError` and `BlockchainError` with typed variants for every failure mode in the chain storage and state management layers.
- Proposed workspace-level lint configuration in `Cargo.toml` to guard against future regressions.
- Learned (the hard way) that replacing `unwrap()` with `?` is never mechanical, sometimes the existing error handling encodes domain-specific invariants.

The midterm plan listed the remaining modules in dependency order:

| Weeks | Modules |
|-------|---------|
| 7–8 | `kv_database.rs`, `lib.rs` (floresta-watch-only), `address_man.rs`, `user_req.rs` |
| 9–10 | `peer_man.rs`, `chain_selector_ctx.rs`, `running_ctx.rs`, `electrum_protocol.rs` |
| 11–12 | All `json_rpc/` files (6 modules), final cleanup |

Here is what actually happened.

## Phase 1: The Wallet Layer (`floresta-watch-only`)

### Preserving Source Errors in the Database Backends

Floresta's watch-only wallet has two storage backends: `kv_database.rs` (persistent, backed by `kv`) and `memory_database.rs` (in-memory, used for testing). Both backends previously swallowed their errors into opaque strings:

```rust
// Before: the actual kv::Error is lost
Err(WatchOnlyError::Other(format!("kv error: {e}")))
```

The fix was to preserve the source error through the chain. The `kv_database` backend now wraps its storage errors with full context:

```rust
// After: source chain is preserved, caller can inspect
Err(WatchOnlyError::Database {
    source: Box::new(e),
})
```

### Typed Lock Accessors

Both backends use `RwLock` to protect shared state. The original code used `.unwrap()` on lock acquisition, which panics if the lock is poisoned (i.e., a thread panicked while holding it). I introduced typed accessor methods that convert lock poisoning into a recoverable error:

```rust
fn read_lock(&self) -> Result<RwLockReadGuard<'_, Inner>, WatchOnlyError> {
    self.inner.read().map_err(|_| WatchOnlyError::Internal(
        "lock poisoned - a thread panicked while holding this lock".into()
    ))
}
```

Every method that previously called `.read().unwrap()` or `.write().unwrap()` now calls `self.read_lock()?` or `self.write_lock()?`. This is a pattern I ended up reusing across several modules.

## Phase 2: The Wire Layer (`floresta-wire`)

The wire layer is where Floresta talks to the Bitcoin P2P network. It is the most complex crate — six interconnected sub-modules that handle peer discovery, connection management, chain selection, block synchronization, and user requests. This was the most architecturally interesting part of the project.

### One Error Type Per Sub-Module

The issue's [Standard](https://github.com/getfloresta/Floresta/issues/1053) says: "Each refactored module owns at least one error type." For `floresta-wire`, I created five sub-module error types that all roll up into a single `WireError`:

```rust
pub enum WireError {
    AddressMan(AddressManError),
    PeerMan(PeerManError),
    ChainSelector(ChainSelectorError),
    UserReq(UserReqError),
    RunningCtx(RunningCtxError),
    Io(io::Error),
    Channel(String),
    // ...
}
```

This means a caller interacting with the wire layer gets a single error type (`WireError`) but can drill down into the specific sub-module that failed. Each sub-module error is designed to tell the caller something *actionable*.

### Address Book: Errors That Help Operators

The address book (`address_man.rs`) manages peer discovery, DNS seeds, stored peer files, and address selection. Its error type is a good example of the "variants must inform the caller" principle:

```rust
pub enum AddressManError {
    PeerFile {
        path: PathBuf,
        source: std::io::Error,
    },
    CorruptedPeerFile {
        path: PathBuf,
        source: serde_json::Error,
    },
    DnsSeed {
        seed: String,
        reason: String,
    },
}
```

A `PeerFile` error tells the operator exactly which file failed and what the I/O error was. A `CorruptedPeerFile` tells them the file exists but cannot be decoded, deleting it makes the node fall back to DNS seeds. A `DnsSeed` error is not fatal on its own since the node tries the remaining seeds.

Notice what is *not* in this enum: lock contention, channel send failures, or internal state corruption. Those are implementation details that collapse into the parent `WireError::Channel` variant, following the issue's principle that internal noise should not become public variants.

### Chain Selection: Where Errors Actually Matter for Consensus

The chain selector (`chain_selector_ctx.rs`) decides which chain tip to follow. Getting this wrong means your node follows the wrong chain. The error type here needed to be especially precise:

```rust
pub enum ChainSelectorError {
    Chain(BlockchainError),
    Io(io::Error),
    Peer(PeerManError),
    // ...
}
```

The critical design decision was wrapping `BlockchainError` (from `floresta-chain`) at the module boundary. This means when a caller sees `WireError::ChainSelector(ChainSelectorError::Chain(..))`, they know the chain error came from the chain selection path specifically, not from peer management or block syncing, which might also produce chain errors in different contexts. This is the "foreign errors are wrapped at the module boundary" principle from issue #1053.

### Guarding Invariants in Transport and Encoding

The transport layer (`transport.rs`, `onion.rs`, `network_message_ext.rs`) handles raw Bitcoin P2P message encoding. These files are full of byte-level operations, indexing into buffers, arithmetic on message lengths, parsing onion addresses. Every `buffer[i]` is a potential panic if the index is out of bounds.

I replaced direct indexing with `.get()` and wrapped arithmetic in checked operations:

```rust
// Before: panics if buffer is too short
let version = buffer[0];

// After: returns an error the caller can handle
let version = *buffer.get(0).ok_or(WireError::Io(
    io::Error::new(io::ErrorKind::InvalidData, "buffer too short")
))?;
```

For arithmetic, I used Rust's `checked_add`, `checked_sub`, and `checked_mul` to prevent silent overflow:

```rust
// Before: silently wraps on overflow
let total = base + offset;

// After: explicit overflow check
let total = base.checked_add(offset).ok_or(WireError::Io(
    io::Error::new(io::ErrorKind::InvalidData, "length overflow")
))?;
```

## Phase 3: The Electrum Layer

`floresta-electrum` bridges the gap between Floresta's internal chain state and Electrum-compatible clients. It is a high-level module that depends on `floresta-chain`, `floresta-watch-only`, and `floresta-wire`.

The main challenge here was that the Electrum protocol handler previously used `unwrap()` when interacting with the chain backend, assuming that chain queries always succeed. In reality, they can fail, the chain might not be synced yet, the database might be temporarily unavailable, or a block hash might not exist.

I pinned the chain error type and propagated protocol failures through a unified error path, so an Electrum client receives a proper JSON error response instead of the server crashing.

## Phase 4: The JSON-RPC Layer (`floresta-node`)

The JSON-RPC layer is Floresta's external interface, what wallets and applications talk to. It has six files, each handling a different aspect of the RPC interface: request parsing, response formatting, blockchain queries, network management, control commands, and the HTTP server itself.

### Request Parsing Gets Its Own Error Type

Previously, invalid RPC requests (missing parameters, wrong types, malformed JSON) were handled with ad-hoc string errors or unwraps. I introduced `JsonRpcError` with specific variants:

```rust
pub enum JsonRpcError {
    MissingParameter(String),
    InvalidParameterType(String),
    InvalidParameterStructure(String),
    InvalidJsonRpcVersion,
    InvalidVerbosityLevel,
    TxNotFound,
    BlockNotFound,
    Chain(String),
    InvalidRequest,
    MethodNotFound,
    // ...
}
```

Each variant maps to a specific JSON-RPC error code, so clients get meaningful error responses instead of generic "internal server error" messages.

### Chain Diagnostics in RPC Errors

One interesting design decision was the `Chain(String)` variant. The chain backend is generic over its error type (it uses a trait), so we cannot structurally embed the concrete chain error. Instead, the diagnostic message is preserved as a string, following the same convention as `ChainstoreError::Internal`. This is a pragmatic compromise, the caller gets a useful error message, and the generic architecture is preserved.

### Test Fixtures

I added a shared `RpcImpl` test fixture (`test_fixture.rs`) that constructs a fully wired RPC handler with mock backends. This makes it easy to write propagation tests for every RPC endpoint without duplicating setup code.

## Phase 5: Metrics, florestad, and the Periphery

### Metrics: Log Instead of Panic

The metrics server previously used `.unwrap()` when binding to a TCP port. If the port was unavailable, the entire node crashed, even though metrics are completely optional. I replaced this with logging:

```rust
// Before
let listener = TcpListener::bind(addr).await.unwrap();

// After
let listener = match TcpListener::bind(addr).await {
    Ok(l) => l,
    Err(e) => {
        tracing::error!("metrics server failed to bind to {addr}: {e}");
        return;
    }
};
```

### florestad: Propagating Startup Failures

The main `florestad` binary is where everything comes together. It previously used `unwrap()` when setting up the Tokio runtime, daemonizing the process, and initializing the node. I replaced these with proper error propagation through `main()`, so startup failures produce a clear error message instead of a panic backtrace.

### Examples and Fuzz Targets

Examples and fuzz harnesses legitimately need `unwrap()`, they are not production code. I added targeted `#![allow]` attributes with `reason` strings explaining why:

```rust
#![allow(
    clippy::unwrap_used,
    clippy::expect_used,
    clippy::panic,
    reason = "fuzz harnesses intentionally crash on unexpected inputs"
)]
```

## Phase 6: Flipping the Switch - `warn` to `deny`

The final step was changing the workspace lint configuration from `warn` to `deny` for all 11 error-handling lints. This is the commit that proposes making the overhaul permanent once merged:

```toml
[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "deny"
panic = "deny"
unreachable = "deny"
todo = "deny"
unimplemented = "deny"
indexing_slicing = "deny"
arithmetic_side_effects = "deny"
map_err_ignore = "deny"
wildcard_enum_match_arm = "deny"
result_unit_err = "deny"
```

With these set to `deny`, any future contributor who tries to add an `unwrap()`, a direct `panic!()`, an unchecked array index, or a `map_err(|_| ...)` will get a **compiler error**, not just a warning. The codebase cannot regress.

## The Hardest Problem in the Second Half

The hardest challenge was not any single module, it was the interaction *between* modules. When you change an error type in `floresta-chain`, every module that depends on it needs to update its `From` implementations, its match arms, and its tests. When you add a new variant to `WireError`, every handler that matches on it needs a new arm.

The tracking issue prescribed a strict dependency order, low-level modules first, so high-level modules never need fixups for upstream changes. This worked well in theory, but in practice I found myself going back to earlier modules as I understood the full picture better. For example, after designing the wire layer's error hierarchy, I realized that `BlockchainError` needed a `Chain(String)` variant to carry diagnostic context through generic trait boundaries. That change touched `floresta-chain` even though I had "finished" it weeks earlier.

The lesson: in a large refactor, you are not done with a module until *every module that depends on it* is also done.

## What's Next: The Gradual Review Process

Because this refactor touches 86 files and changes the return types of core consensus and wire functions, merging it all at once into the `getfloresta/Floresta` repository would be extremely risky. 

Instead, I have staged all 41 commits in a [consolidated PR on my personal fork](https://github.com/Vedd-Patel/Floresta/pull/5). This allows my mentor and the maintainers to review the entire error-handling story from top to bottom. To actually upstream these changes, we will cherry-pick the commits and open a single, focused PR for each module. This step-by-step approach ensures we do not introduce subtle bugs or performance regressions into the main repository.

## What I Learned in the Second Half

### Error Hierarchies Are API Design

Designing error types is not a cleanup task, it is API design. The shape of your error enum defines what callers can and cannot distinguish. If you put lock poisoning and network timeouts in the same variant, callers cannot tell them apart. If you make every internal failure its own variant, your enum becomes too noisy to be useful.

The issue's principle - "variants must inform the caller; a variant exists to tell the user something they can act on" - was the most important design guide. Internal noise collapses into `Internal`; actionable failures get their own variants.

### `map_err` Is a Code Smell

The issue explicitly forbids `map_err` in production error handling. At first I thought this was overly strict, but after seeing how `map_err(|_| MyError::Other("something failed".into()))` throws away the entire source chain, I understand why. Every time you `map_err`, you are deciding that downstream consumers do not need to know what actually went wrong. That is almost never the right call. Instead we use `From` implementations that preserve the source.

### Lint Enforcement Is a Force Multiplier

Setting lints to `warn` is documentation. Setting them to `deny` is enforcement. The difference matters: with `warn`, developers can (and do) ignore warnings. With `deny`, the code does not compile until the issue is fixed. This is especially important in a project with multiple contributors, the lint configuration becomes a living style guide that the compiler enforces.

## Acknowledgments

Huge thanks to my mentor [Joao Leal (jaoleal)](https://github.com/jaoleal) for the detailed code reviews, the design guidance on the tracking issue, and for pushing me to think about error types as API contracts rather than cleanup tasks. His feedback consistently made the code better, from catching naming issues (using `ChainWorkOverflow` for non-chain-work arithmetic) to challenging the design of test suites (do not test that stubs return stubs).

Thanks also to the [Summer of Bitcoin](https://www.summerofbitcoin.org/) program for making this possible, and to the Floresta community for being welcoming to a first-time contributor working on foundational infrastructure.

## Links to My Work

- **Tracking issue:** [Systematic Error Handling Overhaul](https://github.com/getfloresta/Floresta/issues/1053)
- **Final PR:** [Complete the systematic error handling overhaul across all modules](https://github.com/Vedd-Patel/Floresta/pull/5)
- **PR #1075:** [Refactor error handling in flat_chain_store.rs](https://github.com/getfloresta/Floresta/pull/1075)
- **PR #1133:** [Improve error handling in chain_state.rs](https://github.com/getfloresta/Floresta/pull/1133)
- **PR #1144:** [Configure workspace error-handling lints as a status board](https://github.com/getfloresta/Floresta/pull/1144)
- **Floresta repository:** [github.com/getfloresta/Floresta](https://github.com/getfloresta/Floresta)

The goal was a node that tells you what went wrong instead of crashing. 41 commits later, that is what Floresta does. 🦀⚡
