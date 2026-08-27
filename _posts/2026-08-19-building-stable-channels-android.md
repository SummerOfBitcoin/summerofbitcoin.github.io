---
layout: post
title: "Building a Bitcoin Lightning Wallet for Android from Scratch"
date: 2026-08-19
author: Anurag Thakur
categories: [Lightning-Network, Wallets, Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-08-19-stable-channels-android-header.jpg
---

### Stable Channels: holding dollars on Bitcoin, no bank needed

---

This summer I built the Android app for [Stable Channels](https://github.com/toneloc/stable-channels). The idea behind the project is simple: you open a Lightning channel, and instead of your balance going up and down with Bitcoin's price, it stays fixed in dollars. Two nodes settle the difference between themselves every minute. No stablecoin issuer, no bank, just math.

The iOS and desktop versions already existed. My job was Android.

---

### Starting from nothing

When I joined, the Android code was basically a placeholder. There was a Compose scaffold, a few empty service files, and nothing else. No working UI, no node integration, nothing a user could actually open.

I'd never worked with Lightning before. The first two weeks I just read — the LDK docs, the iOS Swift code, some Rust. I didn't write any Android code until I felt I understood what the node was actually doing. That slow start paid off later.

---

### What I shipped

**Sending and receiving**

This was the core of the app. Users needed to send and receive both over Lightning and on-chain. I built both flows and redesigned the amount input to be centered and borderless — matching the iOS design. I also added a Send Max button, which calculates the amount after fees so you can empty your wallet in one tap.

[PR #114](https://github.com/toneloc/stable-channels/pull/114) — send and receive flows
[PR #216](https://github.com/toneloc/stable-channels/pull/216) — show the actual routing fee on the send screen

**Trading BTC ↔ USD**

The Buy and Sell screens are what make this wallet different. Tap Buy, enter a dollar amount, confirm — and from that point your balance holds that dollar value even if BTC moves. Sell unwinds the agreement and gives you BTC back.

[PR #111](https://github.com/toneloc/stable-channels/pull/111) — terminology and labels
[PR #113](https://github.com/toneloc/stable-channels/pull/113) — trade screen design

**Keeping the peg stable**

Every 60 seconds the app checks if the BTC price moved enough to matter. If it did, it sends a small Lightning payment to settle. I spent a lot of time on the failure cases — what if price feeds are all down? What if the node is still syncing? What if a payment just arrived? Each one needed a separate fix.

[PR #76](https://github.com/toneloc/stable-channels/pull/76) — price staleness check
[PR #139](https://github.com/toneloc/stable-channels/pull/139) — stability fallback when backing sats is zero
[PR #35](https://github.com/toneloc/stable-channels/pull/35) — unit tests (20 tests, all passing)

**QR scanner**

I built it with CameraX and ML Kit. It scans invoices and addresses live from the camera. I also added photo picker support so users can scan from a screenshot — turns out people share invoices as images a lot.

[PR #82](https://github.com/toneloc/stable-channels/pull/82) — QR scanner
[PR #211](https://github.com/toneloc/stable-channels/pull/211) — fixed gallery scanning and an ANR on background stop

**Biometric auth**

Sending money, viewing the seed phrase, switching the LSP — all require fingerprint or face unlock. If the device doesn't support biometrics, it falls back gracefully.

[PR #96](https://github.com/toneloc/stable-channels/pull/96) — biometric auth and clipboard security
[PR #106](https://github.com/toneloc/stable-channels/pull/106) — auth gates on all sensitive actions

**Price chart**

There's a 30-day BTC/USD chart on the home screen. I added a Kraken backfill at startup so it shows real data from day one instead of "Collecting price data..." for two weeks. It supports 1D, 1W, and 1M views with a draggable tooltip.

[PR #72](https://github.com/toneloc/stable-channels/pull/72) — Kraken hourly backfill
[PR #80](https://github.com/toneloc/stable-channels/pull/80) — skip backfill when data already exists

**Background service**

The LDK node has to stay alive in the background — otherwise you miss incoming payments and stability checks stop running. I hooked it up to Firebase push notifications so the service wakes up, does what it needs to do, and goes back to sleep.

There was a weird issue though. Every time a user left the app — even just to reply to a message — the system treated it as a full app exit and restarted the node. That's expensive and unnecessary. I added a 60-second grace period so it only stops if the user is actually gone for a while.

[PR #148](https://github.com/toneloc/stable-channels/pull/148) — background grace period
[PR #116](https://github.com/toneloc/stable-channels/pull/116) — FCM hardening

**Settings**

I rebuilt settings from a flat list into a navigation hub with 10 sub-screens — LSP, backup, diagnostics, network, and more.

[PR #83](https://github.com/toneloc/stable-channels/pull/83) — settings navigation

**Custom LSP**

Hardest PR of the summer. LDK bakes the LSP's pubkey and address into the node at build time, so switching LSPs means rebuilding the whole node. I built a settings screen for it with validation, a channels-open warning, and auto-rollback — if the new LSP doesn't connect, the app reverts to the previous one and restarts without leaving the node in a broken state.

[PR #201](https://github.com/toneloc/stable-channels/pull/201) — custom LSP configuration

**Logs and diagnostics**

I added a screen that exports a ZIP of the audit and LDK logs. I also changed how channel close events are recorded — structured fields instead of a raw debug string — so force-closes are actually debuggable. I added this to iOS and the Rust desktop backend in the same PR.

[PR #191](https://github.com/toneloc/stable-channels/pull/191) — logs and diagnostics

**Real-time transaction updates**

I added Mempool websocket support. When a transaction hits the mempool, the balance updates on screen immediately instead of waiting for the next polling cycle.

[PR #222](https://github.com/toneloc/stable-channels/pull/222) — mempool websocket

**Android 16 compliance**

I bumped the target SDK to API 36 and fixed the foreground service type — required by Android 14+ and Google Play's deadline.

[PR #199](https://github.com/toneloc/stable-channels/pull/199) — API 36 compliance

---

### One technical decision worth explaining

For the custom LSP feature I had three options: restart the whole Android process, restart just the Activity, or do an in-process `NodeService.stop()` / `start()`. The process restart would have closed the database ungracefully. The Activity restart left background jobs running against the old node config. In-process restart was more work but the safest — it reused the existing lifecycle and didn't touch anything it shouldn't.

---

### What I learned

I thought I knew why Bitcoin wallets were hard. I didn't.

Everything in a wallet is stateful. Sending, receiving, opening a channel, closing one — each of these touches the LDK database, the app database, the UI, and the audit log at the same time. If any one of those gets out of sync, the user sees wrong data. Or worse — they think a payment worked when it didn't.

The bug that stuck with me most was in the swap flow. The app was calling the Mempool API to check if a transaction confirmed. Every call came back 400. I spent hours on it. Turned out the txid had `:0` appended — the output index — making the URL invalid. That single character locked the UI in "Swap in progress" forever. The fix was one line. Finding it took a day. That's wallets.

The other thing I learned: reading code is real work. I spent two weeks reading iOS Swift before I wrote any Android. That wasn't procrastinating — that was the job.

---

### What remains

The app is live on the Play Store. Next up for me is seed phrase encryption — protecting the mnemonic at rest so the keys need an explicit user action to decrypt.

---

### Thanks

Thanks to Tony for being a good mentor and for trusting me to own the Android side. And to Summer of Bitcoin for the push.

All my PRs: [github.com/toneloc/stable-channels/pulls](https://github.com/toneloc/stable-channels/pulls?q=is%3Apr+is%3Amerged+author%3ADevAnuragT)
