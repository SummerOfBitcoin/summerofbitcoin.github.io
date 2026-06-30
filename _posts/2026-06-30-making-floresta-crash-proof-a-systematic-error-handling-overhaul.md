---
layout: post
title: "Making Floresta Crash-Proof: A Systematic Error Handling Overhaul"
date: 2026-06-30
author: Ved Patel
categories: [Development, Open-Source, Bitcoin-Core]
image: ../assets/images/blog_content/floresta-error-handling.png
---

A Bitcoin full node is not a web app where you can show the user a "something went wrong" page and try again. It validates a chain that represents real monetary value. An uncontrolled crash during block validation forces the node offline, which can lead to potential downtime for applications depending on it, or even hours of re-syncing if persisted state was damaged. So what happens when your node's codebase is full of code like this?

```rust
let header = chainstore.get_header(&hash).unwrap();
```

That single `.unwrap()` looks harmless, but it is a ticking time bomb. If the header is not found - maybe the database got corrupted, maybe a disk write failed halfway - the entire node crashes. While a Rust panic emits a generic panic message, it is an uncontrolled crash, not typed, recoverable error handling. No graceful recovery. Just gone.

My Summer of Bitcoin project is about fixing exactly this across [Floresta](https://github.com/getfloresta/Floresta), a lightweight Bitcoin full node written in Rust.

## Who I Am and What I Am Working On

I am Ved Patel, a contributor to Floresta through Summer of Bitcoin 2026, mentored by [Joao Leal (jaoleal)](https://github.com/jaoleal). Floresta uses [Utreexo](https://eprint.iacr.org/2019/611.pdf) - a novel accumulator that lets you verify the entire Bitcoin blockchain without storing every unspent transaction output. This makes running a full node practical on resource-constrained devices like a Raspberry Pi, which matters for Bitcoin's decentralization goals.

My project is a systematic error handling overhaul tracked in [issue #1053](https://github.com/getfloresta/Floresta/issues/1053). The goal: replace every crash-prone `unwrap()`, `expect()`, and `assert!()` across 16 modules with typed, recoverable error handling - and enforce it with compiler-level lint rules so the problem never comes back.

## What Problem My Project Is Solving

Floresta's codebase had two related problems.

**First, crash-prone error handling.** Functions used `unwrap()` and `expect()` on operations that could legitimately fail at runtime - disk reads, hash lookups, arithmetic on block heights. If any of these failed, the node crashed instead of recovering gracefully.

**Second, uninformative errors.** Even where errors were handled, they often looked like this:

```rust
pub enum ChainstoreError {
    Other(String),  // everything is a string
}
```

A caller receiving `Other("something went wrong")` cannot distinguish between a corrupted database and a missing header. It cannot decide whether to retry, reindex, or shut down. The error tells you nothing actionable.

The fix is to build a **typed error model** where every failure is matchable (callers can distinguish specific failures), traceable (the source chain is preserved end-to-end), and enforced (lint rules prevent regressions).

Here is what that looks like in practice:

```rust
pub enum FlatChainstoreError {
    Io(io::Error),
    HeaderNotFound,
    FullIndex,
    BadMagic(u32),
    UnsupportedSchema(u32),
    CorruptedDatabase,
}
```

Now upstream code can react differently to different failures:

```rust
match store.get_header(&hash) {
    Err(FlatChainstoreError::HeaderNotFound) => { /* try a fallback */ },
    Err(FlatChainstoreError::CorruptedDatabase) => { /* trigger reindex */ },
    Err(e) => return Err(e.into()),
    Ok(header) => { /* proceed */ },
}
```

That is the difference between "your node crashed" and "your node noticed a problem and handled it."

## What I Have Worked On in the First Six Weeks

### PR #1075 - On-Disk Block Storage (`flat_chain_store.rs`)

This was the first and most challenging module. `FlatChainStore` persists block headers, fork data, and Utreexo accumulators to disk using memory-mapped files. It is full of unsafe code, open-addressing hash maps stored in raw byte arrays, and subtle invariants around uninitialized memory.

In this open PR, I proposed `FlatChainstoreError` with specific variants for every failure mode, replacing every `unwrap()` and `expect()` with proper `Result` propagation, and writing error propagation tests that verify each error path by matching on the variant - not by inspecting display strings.

### PR #1133 - Chain State Manager (`chain_state.rs`)

This is the heart of Floresta - the module that validates block headers, manages the best chain, handles reorganizations, and maintains the Utreexo accumulator. It had `assert!()` calls that would crash the node if the chain state did not match expectations, a `find_best_chain()` function that panicked instead of returning an error, and arithmetic operations that silently wrapped on overflow.

In this open PR, I proposed making `find_best_chain()` and `ChainState::new()` return `Result`, replacing raw arithmetic with `checked_add`, and introducing new `BlockchainError` variants like `ChainNotInitialized`, `InvalidTip`, `Overflow`, and `Unsupported`. A reviewer correctly pointed out that I was using `ChainWorkOverflow` for non-chain-work arithmetic - a naming issue I fixed by adding a separate `Overflow(&'static str)` variant. That kind of feedback is exactly why code review matters.

### PR #1144 - Workspace-Level Lint Configuration

Instead of adding lint blocks to every single file, I proposed configuring the lint rules at the workspace level in `Cargo.toml`. This means the entire crate can be guarded by rules like:

```toml
[workspace.lints.clippy]
unwrap_used = "warn"
expect_used = "warn"
panic = "warn"
indexing_slicing = "warn"
arithmetic_side_effects = "warn"
```

Any future contributor who tries to add an `unwrap()` will get a compiler warning, helping to catch these issues early before code review.

## The Hardest Problem I Faced

The hardest challenge was not writing error types - it was understanding the code deeply enough to know which errors were even possible.

In `flat_chain_store.rs`, there is a function called `hash_map_find_pos` that probes an open-addressing hash map stored in a memory-mapped file. It has a subtle edge case: when a slot holds index zero (which is also what uninitialized memory looks like), the function needs to distinguish between "this slot is empty" and "this slot genuinely holds the genesis block at position 0."

The original code handled this by catching a `HeaderNotFound` error and treating it as an empty slot - using error handling as control flow. When I tried to replace this with clean `Result` propagation using `?`, genesis block lookups broke because the error would propagate upward instead of being caught locally.

I had to trace through the probing logic carefully to understand the invariant: the genesis block's hash is always checked first in the comparison, so the empty-slot fallback is never reached for index zero in practice. This taught me that replacing `unwrap()` with `?` is never a mechanical find-and-replace - sometimes the existing error handling encodes domain-specific invariants that you need to fully understand before refactoring.

## What I Learned

### Rust's Error Architecture in Large Codebases

Before this project, I thought error handling was mostly about choosing between `Result` and `panic!()`. Working on Floresta taught me the architecture of error handling - specifically, the pattern of defining a `DatabaseError` trait, having each module implement its own concrete error enum, and using a blanket `impl<T: DatabaseError> From<T> for BlockchainError` to let errors propagate through `?` while erasing the concrete type at module boundaries.

This lets the chain state manager work with any storage backend without knowing its specific error types, while still giving the storage backend full freedom to define detailed, domain-specific errors internally.

### Commit Structure Matters as Much as Code Quality

My first PR was a single monolithic commit with 300+ lines of changes. The reviewer's feedback was essentially: "This is too much to review at once." I learned to split refactors into small, logically grouped commits - one for introducing error types, one for migrating the module, one for updating call sites, one for tests. Each commit compiles and passes tests independently.

This sounds obvious in hindsight, but it changes how you think about writing code. You are not just writing for correctness - you are writing for reviewability.

### Bitcoin Node Reliability Is Non-Negotiable

Working on error handling has given me a deep appreciation for what "production-grade" means in Bitcoin software. Imagine you are running Floresta on a Raspberry Pi in a remote location, validating the Bitcoin blockchain. A momentary disk hiccup during a write, and if your node uses `unwrap()`, it is dead. You have to restart and recover, and hope it does not happen again. Every `unwrap()` I replace is a potential crash that someone running a node will never experience.

## What I Plan to Finish Before the Final Evaluation

The first half covered the two largest modules in `floresta-chain`. The second half follows the dependency order defined in the tracking issue - low-level modules first, so high-level modules never need fixups for upstream error-type changes:

| Weeks | Modules |
|-------|---------|
| 7–8 | `kv_database.rs`, `lib.rs` (floresta-watch-only), `address_man.rs`, `user_req.rs` (floresta-wire) |
| 9–10 | `peer_man.rs`, `chain_selector_ctx.rs`, `running_ctx.rs`, `electrum_protocol.rs` |
| 11–12 | All `json_rpc/` files (6 modules), final cleanup |

By the final evaluation, every major module in Floresta will have typed error models, preserved source chains, propagation tests, and lint guards preventing regression. The end result is a node that tells you what went wrong instead of crashing - which is exactly what you want from software that handles real money.

## Links to My Work

- **Tracking issue:** [#1053 - Systematic Error Handling Overhaul](https://github.com/getfloresta/Floresta/issues/1053)
- **PR #1075:** [Refactor error handling in flat_chain_store.rs](https://github.com/getfloresta/Floresta/pull/1075)
- **PR #1133:** [Improve error handling in chain_state.rs](https://github.com/getfloresta/Floresta/pull/1133)
- **PR #1144:** [Configure workspace-level clippy lint rules](https://github.com/getfloresta/Floresta/pull/1144)
- **Floresta repository:** [github.com/getfloresta/Floresta](https://github.com/getfloresta/Floresta)

Until the next block is mined, happy coding! 🦀⚡