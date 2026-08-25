---
layout: post
title: "Building an Android CoinSwap Taker: My Summer of Bitcoin Final Report"
date: 2026-08-22
author: Tanishka Nibariya
categories: [Wallets, Privacy, Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-08-22-android-coinswap-taker-final-report.jpg
---

Running a multi-maker coinswap from a phone sounds simple until Tor, UniFFI, and Android lifecycle all show up at once. This is my Summer of Bitcoin 2026 final report for [Citadel Tech](https://github.com/citadel-tech) / [Citadel FOSS](https://github.com/citadel-foss): how I built a Jetpack Compose **CoinSwap taker** that talks to the official Rust stack, discovers makers over Orbot, and completes real swaps on a physical device.

If you care about Bitcoin privacy tooling on mobile, or about wiring Rust protocol code into Android without rewriting it, this post walks through what I set out to do, what shipped, what is still open, and what I would do differently next time.

**Pull request (submission):** [citadel-foss/coinswap-kotlin#2](https://github.com/citadel-foss/coinswap-kotlin/pull/2)  
**Branch / fork:** [tanishkaa08/coinswap-kotlin (`ffi-rpc`)](https://github.com/tanishkaa08/coinswap-kotlin/tree/ffi-rpc)

---

## What I set out to do

CoinSwap already had a working desktop taker flow. The gap was mobile: there was no first-class Android client that could:

- restore / unlock a taker wallet on-device
- sync balances and UTXOs
- discover makers over Tor
- prepare and execute Taproot or Legacy coinswaps
- recover from interrupted swaps
- match the mental model of the desktop taker without reimplementing protocol logic in Kotlin

My goal was not a demo UI on top of stubs. I wanted a phone app that uses the **same Rust taker** via generated UniFFI bindings (`libcoinswap_ffi.so`), so protocol correctness stays in core-lib while Android owns UX, lifecycle, Tor plumbing, and local testing tooling.

---

## What I built

I shipped a full Compose taker app with this stack:

```text
Compose UI
  → ViewModels
    → CoinswapRepository
      → Generated UniFFI Kotlin (org.coinswap)
        → libcoinswap_ffi.so
          → Rust Taker
            → Electrum (electrs) + Tor (Orbot SOCKS)
```

### App surfaces

- **Login / setup** — wallet passcode and session restore; Electrum backend for Citadel signet or local regtest
- **Home** — balances, sync, Tor status, UTXO list with privacy cues
- **Receive / send** — ordinary wallet spends and QR receive
- **Markets** — offerbook sync and live maker list over Tor
- **Swap** — Taproot or Legacy, amount, maker count, onion filtering, coin control (auto by default, manual opt-in)
- **Swap progress** — foreground service + notification while a swap runs; phase text from `swap_tracker.cbor`
- **Recovery** — path for interrupted / failed taker swaps
- **History / reports** — wallet transactions and local coinswap reports
- **Settings** — Tor / Orbot helpers, reconnect, wallet data reset

### Backend glue

- `CoinswapRepository` wraps UniFFI `Taker` (init, sync, offers, prepare/start/recover, reports, sends)
- `TakerHolder` owns the live native instance with lease-aware close and FFI mutexes
- Activity-scoped swap ViewModel + `SwapForegroundService` so an in-flight swap survives tab switches and backgrounding
- Config / Tor helpers aligned with the desktop taker model (Orbot on `127.0.0.1:9050`)

### Regtest / phone tooling

Under `scripts/` I added helpers for local phone testing: WSL maker bring-up, Tor SOCKS/control portproxy on Windows, `adb reverse` for connectivity, and quick probes. I exercised multi-maker coinswaps on a physical phone against Docker makers on regtest.

---

## Major technical / design decisions

### 1. UniFFI over rewriting the protocol

The important boundary: **wallet and swap logic stay in Rust**. Kotlin is an adapter. That kept the Android app honest to upstream coinswap behavior and avoided a second, divergent protocol implementation.

### 2. Electrum backend instead of Bitcoin Core RPC on the phone

Early drafts mirrored a Core RPC + ZMQ desktop setup. For a real mobile path we moved to **Electrum (`electrs`)** via `BackendConfig`. Clearnet Electrum connects directly; maker discovery and swaps still go through Orbot. That matches how a phone should talk to the chain without running bitcoind on-device.

### 3. Orbot-only Tor — remove in-app Tor

I initially explored embedded Tor. In practice it dropped proof-of-funding mid-swap (`UnexpectedEof` / SOCKS timeouts) and fought Orbot for port `9050`. The durable choice: **no in-app Tor daemon**. Probe Orbot’s SOCKS on `127.0.0.1:9050`, optionally auth on the control port, and fail clearly when Orbot is not ready.

### 4. Lease-safe native lifecycle

`Taker` is a scarce native resource. Concurrent ViewModels (wallet sync, markets, swap) can race init/close. `TakerHolder` uses:

- `withLeased` so logout / reconnect cannot `close()` a Taker still in use
- `initMutex` / `ffiMutex` / `longOpMutex` so init, short reads, and long Tor/swap ops do not interleave unsafely

### 5. Foreground service for swap execution

Coinswaps are long-running. Keeping execution only in a Compose-scoped coroutine was fragile. A foreground service runs `startCoinswap`, publishes progress on `SwapExecutionBus`, and polls the core-lib tracker file so the UI and notification stay in sync when the user leaves the Swap tab.

### 6. Honest Tor / estimate UI

Configurable SOCKS fields that the native stack ignored were removed or fixed. Offerbook sync failures gate swap prep. Maker estimates reject empty / ineligible selections instead of inventing fake fees. On a money app, that honesty matters more than polish.

---

## What shipped

Delivered in [PR #2](https://github.com/citadel-foss/coinswap-kotlin/pull/2):

| Area | Status |
| --- | --- |
| Compose taker UI (login → home → markets → swap → history → settings) | Shipped |
| UniFFI `Taker` integration (Electrum + Orbot) | Shipped |
| Multi-maker coinswaps on physical phone + regtest makers | Tested end-to-end |
| Wallet sync, receive, send, recovery paths | Exercised in the same setup |
| Swap foreground service + progress tracking | Shipped |
| Local scripts for makers / Tor / adb reverse | Shipped |
| Security and correctness hardening from review (leases, path validation, cancellation, session clearing, backup excludes) | Iterated across review rounds |

The PR title and description summarize the first cut; later commits simplified the UI, removed in-app Tor, and hardened FFI concurrency and swap gates.

---

## What remains

Still open or intentionally deferred:

- **Merge + polish** — the PR is still open; remaining review nits (fee display edge cases, packaging) can land after maintainer feedback
- **Keystore-backed secrets** — move remaining sensitive prefs fully onto Android Keystore
- **Product packaging** — branded package name, release signing, clearer first-run Orbot onboarding
- **Mainnet readiness** — more soak testing outside regtest/signet, fee-rate wiring where the native API supports it, tighter recovery heuristics
- **CI** — automated instrumented / contract tests beyond the unit contracts added during review

---

## What I learned

- **FFI is a product surface.** Binding generation, ABIs, JNA load paths, and native close semantics are as much of the app as Compose screens.
- **Mobile Bitcoin is mostly systems work.** Tor readiness, foreground services, coroutine cancellation, and device networking matter as much as the swap form.
- **Upstream alignment beats clever rewrites.** Mirroring the desktop taker’s Electrum + Orbot model removed entire classes of failure.
- **Review is part of the build.** Iterating on mentor and automated review feedback forced leases, honest Tor probes, and safer session handling I would have deferred.
- **Privacy UX is concrete.** Coin control defaults, UTXO privacy cues, and not claiming Tor when Orbot is not up are user-trust features, not cosmetics.

---

## What happens next

1. Address remaining review comments and work toward merging [PR #2](https://github.com/citadel-foss/coinswap-kotlin/pull/2) into `citadel-foss/coinswap-kotlin`.
2. Keep the Android taker in sync as [coinswap-ffi](https://github.com/citadel-tech/coinswap-ffi) (Electrum branch) evolves.
3. Continue contributing to Citadel’s coinswap mobile / FFI stack after SoB — especially release hardening, recovery UX, and test coverage.
4. Document a shorter “phone + Orbot + electrs” contributor path so others can reproduce the regtest setup without reverse-engineering my scripts.

---

## Conclusion

Summer of Bitcoin let me turn a desktop-only CoinSwap taker story into something you can hold: an Android client that keeps protocol logic in Rust, privacy routing in Orbot, and UX in Compose. The hardest parts were not the screens — they were lifecycle, Tor honesty, and making UniFFI safe under concurrent ViewModels.

If you want to try the work, follow [PR #2](https://github.com/citadel-foss/coinswap-kotlin/pull/2), skim the README on the `ffi-rpc` branch, and ping me with review notes or repro steps. Contributions, questions, and tough bug reports are all welcome.

**Share / tags:** `#SummerOfBitcoin` `#Bitcoin` `#OpenSource`

Thanks to my mentors and the Citadel / CoinSwap community for the reviews, regtest war stories, and patience while Tor and UniFFI fought me on a real phone.
