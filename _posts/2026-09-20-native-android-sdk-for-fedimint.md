---
layout: post
title: "Building a Native Android SDK for Fedimint: My Summer of Bitcoin Project Report"
date: 2026-09-20
author: Om Swami
categories: [Wallets, Development, Open-Source]
image: ../assets/images/blog_content/2026-09-20-fedimint-android-sdk.png
---

This summer I worked with [Fedimint](https://fedimint.org) on native mobile bindings for the [Fedimint SDK](https://github.com/fedimint/fedimint-sdk). The goal was easy to state: an Android or iOS developer should be able to add Fedimint to a native app the same way they add any other library, with no JavaScript bridge in between. Getting there was harder than planned, and a good part of this post is about why.

## What I set out to do

My proposal, *Native Android (Kotlin) & Swift Bindings for Fedimint SDK*, planned to:

- generate idiomatic Kotlin and Swift bindings for the Fedimint client with [mozilla/uniffi-rs](https://github.com/mozilla/uniffi-rs);
- package them as an Android Archive (`.aar`) for Maven Central and a universal XCFramework for Swift Package Manager;
- automate cross-compilation and testing in CI;
- ship minimal native sample apps and integration docs;
- and, as a stretch goal, support multiple federations from a single app.

Before the program I had upstreamed React Native support for the SDK ([fedimint-sdk#239](https://github.com/fedimint/fedimint-sdk/pull/239)), so I knew the Rust client and the pain of cross-compiling it for mobile. The plan was to build on that foundation.

## Why the plan changed (twice)

The biggest story of my summer isn't a piece of code. It's that the layer I was supposed to wrap changed underneath me twice. Each change was the right call, but each one reset part of the work.

**Plan 1: bindings over the RPC layer.** The React Native SDK reached Rust through `fedimint-client-uniffi`, which exposed the client as an RPC surface: requests go in as strings, responses and events come back through a callback. The first plan was to point Kotlin and Swift at that same layer. It would have worked quickly, but every language would inherit a stringly-typed API and re-implement the same request/response plumbing.

**Plan 2: UniFFI the client directly.** Next we considered annotating the Fedimint client itself with UniFFI and removing the RPC layer. That gets rid of the string boundary, but it pushes internal `fedimint-*` types across the FFI. That couples every SDK release to the upstream release train and leaks module plumbing into app code.

**Plan 3: a high-level Rust SDK that every binding is generated from.** Where we landed is written up in [RFC #344](https://github.com/fedimint/fedimint-sdk/issues/344): a new crate, `rust/fedimint-sdk`, that defines its own types and one ergonomic API. An `Sdk` owns one seed and any number of federations. Each `Federation` exposes `ecash()`, `lightning()` and `onchain()` facades, and every flow returns the same `Operation` handle you can observe, re-attach to after a restart, or await. Errors carry stable codes, and a failed payment is a *state*, not an error. Kotlin, Swift, wasm and React Native all become views of this one API instead of parallel copies that drift apart.

This was the most robust option. It also meant the bindings couldn't be finished until the Rust SDK they bind existed. I owned the planning for the SDK, and all of the UniFFI work that sits on top of it: the FFI layer on the Rust crate and the Android SDK generated from it.

At one point I opened the Android PR early so we'd have something real to test against. My reviewer, rightly, asked me to hold it as a draft until the Rust SDK was complete; otherwise every Rust change would also become binding work and slow the whole effort down. That is the honest reason my timeline slipped. A nice side effect: multi-federation support, a "future" item in my proposal, is now part of the SDK's core design rather than a bolt-on.

## What I built

### 1. An opt-in UniFFI feature for the Rust SDK ([#384](https://github.com/fedimint/fedimint-sdk/pull/384))

I added a `uniffi` cargo feature to `rust/fedimint-sdk`. With it off (the default, and every wasm build) the crate is unchanged. With it on, the crate exports its *own* methods and types across the FFI, with no binding-only glue. The first stage was the bootstrap surface:

```rust
create_fedimint_sdk(data_dir: String, mnemonic: Option<Mnemonic>) -> Result<Arc<Sdk>>

Sdk::export_mnemonic(&self) -> Mnemonic
Sdk::preview(&self, &InviteCode) -> Result<FederationPreview>
Sdk::join(&self, &InviteCode) -> Result<Federation>
```

The rest of the PR was about making that surface production-shaped:

- **A written decision record** (`DECISION.md`) comparing a separate FFI crate with an in-crate feature. We chose the in-crate feature.
- **Build plumbing.** The crate builds as `cdylib` (what Android loads and what `uniffi-bindgen` reads metadata from) and `staticlib` (the iOS form). A size-optimized release profile took the Android library from roughly 82 MB to 61 MB per ABI.
- **`panic = "abort"`.** This is a correctness setting, not a size tweak: a panic must never unwind across the FFI boundary.
- **Android-specific fixes.** 16 KB page alignment, which the Play Store requires for Android 15+ targets. A small build-script stub for an `aws-lc` symbol that Android's linker resolves incorrectly.
- **A tiny host-side `uniffi-bindgen` crate.** Generating bindings no longer compiles the entire Fedimint tree on the host.
- **CI coverage.** Clippy runs with the feature on. A smoke test builds the real library, generates Kotlin from it with the Android config, and checks the output.

### 2. Fixing a runtime crash upstream

While testing on Android, the app aborted with tokio's `can call blocking only when running on the multi-threaded runtime`.

The cause is subtle. UniFFI's tokio support uses `async-compat`, which spins up a *single-threaded* fallback runtime when a call arrives from a Kotlin or Swift thread. Tasks spawned inside that call run on the fallback runtime, and tokio's `block_in_place` aborts on a current-thread scheduler.

I added an opt-in `multi-thread` feature to async-compat ([smol-rs/async-compat#50](https://github.com/smol-rs/async-compat/pull/50)). It was merged and released in **0.2.6**, so the SDK now depends on the released crate instead of my fork. I also proposed enabling it inside UniFFI ([mozilla/uniffi-rs#2992](https://github.com/mozilla/uniffi-rs/pull/2992)). The maintainers would rather let users supply their own runtime, so that PR is still open. It was a good lesson in how upstream maintainers weigh a one-line fix against a process-wide default.

### 3. The Android SDK ([#388](https://github.com/fedimint/fedimint-sdk/pull/388))

This is the main deliverable. It adds an `android/` Gradle project with a library module that becomes the AAR, plus a demo app. There is **no hand-written Kotlin**: everything is generated from the Rust crate's exports.

The build is deliberately split in two:

1. **The native library.** Nix cross-compiles it for `arm64-v8a` and `x86_64`. This step is expensive (RocksDB and aws-lc are built from C), cacheable, and not specific to any language.
2. **The bindings.** They are generated *from the built `.so`*, not from source, so the Kotlin can never describe a different binary from the one the device loads.

CI mirrors that split. One workflow builds the native library once, and the Kotlin workflow generates against that artifact. The costly half is paid for once, and a future Swift generator can start from the same binary.

The PR also expanded the FFI surface from the bootstrap calls to the whole per-federation API:

- federation lifecycle;
- the ecash, Lightning and on-chain facades (`quote` → `send`/`receive`);
- typed operation handles;
- seed recovery, activity history and metadata.

The demo app has one section per SDK call: wallet status with a live balance, join/preview/recover, Lightning invoice and pay, ecash send and redeem, deposit address, on-chain send, and recent activity. I tested it on a Pixel emulator against a live mutinynet federation. After several review rounds, #388 was merged on September 18 with all 22 checks passing.

Using it from Kotlin looks roughly like this (simplified; see `android/app` for the real demo):

```kotlin
val sdk = createFedimintSdk(context.filesDir.path, null) // generates and persists a seed
val invite = InviteCode.parse(inviteString)
val preview = sdk.preview(invite)   // name, network, guardians, modules
val federation = sdk.join(invite)
```

## Design decisions I'd defend

- **Bindings are a view, not a copy.** Exporting the crate's own types means a Rust API change can't silently diverge from Kotlin or Swift.
- **Secrets and bearer values cross as objects, not strings.** `Mnemonic`, `InviteCode` and ecash `Notes` are UniFFI objects. A generated Kotlin data class prints every field in `toString()`, and printing ecash notes hands over spendable value. As objects, their contents only come out through an explicit call.
- **Single-use quotes are enforced at the FFI boundary.** In Rust, `send` takes a quote by value, so reusing it is a compile error. A Kotlin caller only holds a shared reference, so a small guard (`QuoteClaim`) turns a second `send` with the same quote into a clear runtime error. The guard is shared across all three facades.
- **Generic operations are monomorphized with one macro.** UniFFI can't export generic types, so `ffi_operation!` generates one concrete object per `Operation<S>`.
- **Exhaustive projections.** Where an FFI record mirrors a Rust record, it is built through a fully destructured `From` impl. Adding a field in Rust then fails to compile until the binding carries it too.

## What shipped

| Artifact | What it is | Status |
| --- | --- | --- |
| [fedimint-sdk#384](https://github.com/fedimint/fedimint-sdk/pull/384) | Opt-in UniFFI feature for the Rust SDK | Merged |
| [fedimint-sdk#388](https://github.com/fedimint/fedimint-sdk/pull/388) | Android SDK, demo app, CI; full per-federation FFI surface | Merged |
| [fedimint-sdk#366](https://github.com/fedimint/fedimint-sdk/pull/366) | Release pipeline: Changesets CLI v3 / action v2 migration | Merged |
| [fedimint-sdk#353](https://github.com/fedimint/fedimint-sdk/pull/353) | Android E2E tests with Appium against the native demo | Draft |
| [smol-rs/async-compat#50](https://github.com/smol-rs/async-compat/pull/50) | Multi-thread fallback runtime | Merged, released in 0.2.6 |
| [mozilla/uniffi-rs#2992](https://github.com/mozilla/uniffi-rs/pull/2992) | Multi-threaded fallback runtime in UniFFI | Open |

## Reviewing and maintaining

A lot of my summer went into reviewing. I reviewed and merged most of the Rust SDK implementation PRs:

- tracking fedimint master ([#374](https://github.com/fedimint/fedimint-sdk/pull/374));
- value types ([#375](https://github.com/fedimint/fedimint-sdk/pull/375));
- storage, builder and federation lifecycle ([#376](https://github.com/fedimint/fedimint-sdk/pull/376));
- operation infrastructure ([#377](https://github.com/fedimint/fedimint-sdk/pull/377));
- activity history ([#387](https://github.com/fedimint/fedimint-sdk/pull/387)).

I also reviewed and merged contributor PRs: CI flakiness fixes ([#350](https://github.com/fedimint/fedimint-sdk/pull/350)), local CI change detection ([#373](https://github.com/fedimint/fedimint-sdk/pull/373)), the web example's onboarding flow ([#357](https://github.com/fedimint/fedimint-sdk/pull/357)), and cross-platform checksums in the Expo plugin ([#324](https://github.com/fedimint/fedimint-sdk/pull/324)). Reviewing the SDK as it was built kept the Rust API and the bindings moving in step.

## Community

I lead the Fedimint developer call. That covers all the stuff happening in the Fedimint SDK. Announcement: [Fedimint dev call](https://x.com/fedimint/status/2087287915667767792?s=20).

## What remains

- **The Swift/iOS SDK.** XCFramework, Swift Package Manager manifest, and an iOS sample app. The crate already builds a `staticlib` and the UniFFI feature was designed for Swift from the start, so this is next.
- **Publishing.** The AAR is assembled in CI but not yet published to Maven Central.
- **Binding conformance tests** ([#353](https://github.com/fedimint/fedimint-sdk/pull/353)). These cover repeated quote use, subscription completion and cancellation, malformed inputs, and error details, all through the generated bindings.

## What I learned

- **FFI design is API design.** The hard questions weren't "how do I call Rust from Kotlin" but "what happens when Kotlin calls this twice," "what does `toString()` leak," and "who owns this handle."
- **Async runtimes don't compose by default.** Debugging the `block_in_place` abort taught me more about tokio's scheduler than any tutorial. Taking the fix upstream taught me that maintainers weigh defaults that affect every user very differently from local fixes.
- **Cross-compiling C dependencies for Android is its own discipline.** Linker quirks, page alignment and host-vs-target toolchains in Nix all showed up. Caching the expensive step is what makes CI bearable.
- **Scope and sequencing matter as much as code.** Being asked to hold my Android PR until the SDK was ready was frustrating in the moment and correct in hindsight.

## What's next

I plan to keep working on this after the program. The next steps are the Swift SDK, publishing both SDKs, landing the conformance tests, and turning the Android demo into a real wallet.

