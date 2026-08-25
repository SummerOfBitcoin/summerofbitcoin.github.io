---
layout: post
title: "Improve BDK Wallet Balance Interface and Related Methods"
date: 2026-08-23
author: Dmenec
categories: [BDK, Wallets, Development, Open-Source]
image: ../assets/images/blog_content/dmenec.jpeg
---

Hi! I'm Dmenec, a newbie in open source contribution and this summer I've been one of the participants in SoB.

My work was mainly to modify BDK's wallet balance interface and try to solve these issues ([#273](https://github.com/bitcoindevkit/bdk_wallet/issues/273) and [#16](https://github.com/bitcoindevkit/bdk_wallet/issues/16)), which I thought would be easier than it ended up being.  

So, let me introduce a little bit everything.

BDK's `balance` splits funds into four buckets: `confirmed`, `immature`, `trusted_pending` and `untrusted_pending`. The trusted/untrusted split exists to flag unconfirmed coins that a third party could still replace before they confirm.

The old classification decided trust from the output itself: a coin landing on an owned (change) address was trusted, otherwise untrusted. That looks at where the coin *landed* instead of where it *came from*, and it gives two wrong answers:

- Change received inside someone else's unconfirmed transaction was counted as `trusted_pending`.
- A self-send consolidating your own coins onto a receive address was counted as `untrusted_pending`.

## Classification in `bdk_chain`

The fix classifies a coin from its unconfirmed ancestry. In `bdk_chain` ([#2246](https://github.com/bitcoindevkit/bdk/pull/2246)) I added `classify_outpoints`, returning an `Eligibility` per output:

- `Settled`: confirmed given the caller's threshold.
- `Immature`: coinbase not matured yet.
- `Unsettled(Trust)`: unconfirmed, where `Trust` is `Trusted`, `Untrusted` or `Unknown`.

Trust is resolved by an iterative post-order walk over the ancestry, driven by two caller-supplied closures:

- `does_taint(tx)`: the transaction pulls in an output the caller doesn't own.
- `is_settled(pos)`: the chain position counts as settled.

The walk stops at settled ancestors (trusted), short-circuits to `Untrusted` as soon as an ancestor taints, and yields `Unknown` when an ancestor is missing from the canonical view. Visited txids are memoized in a `HashMap<Txid, Trust>`, so an ancestor shared by several outputs is walked once. `balance` is then a `FromIterator` fold over `classify_outpoints`, mapping each `Eligibility` to its bucket.

Now, the solution seems easy but it took several meetings with my mentor nymius and several rounds of reviews to get something to port.  

## Balance in `bdk_wallet`

`Wallet::balance` ([#431](https://github.com/bitcoindevkit/bdk_wallet/pull/431)) now supplies the two closures and lets `CanonicalView` do the walk:

- `does_taint`: an input spends an outpoint whose `script_pubkey` isn't in the wallet's index.
- `is_settled`: built from a new `min_confirmations` argument, so a caller can require N confirmations before a coin counts as settled.

## Timelocked coins

A confirmed coin can still be unspendable when its descriptor carries a timelock that hasn't matured yet. Today the balance counts it as `confirmed`, overstating what you can actually move.

I'm adding a `locked` bucket for these ([#538](https://github.com/bitcoindevkit/bdk_wallet/pull/538), closing [#180](https://github.com/bitcoindevkit/bdk_wallet/issues/180)). A timelock lives in the descriptor, which is a wallet concept and not chain data, so the check stays in the wallet. The logic uses `classify_outpoints` and, for a `Settled` output whose timelock isn't satisfied at the current tip, adds the value to `locked` instead of `confirmed`. Time-based locks are left as a follow-up, as I didn't find an easy way (yet) to check the MTP.

## Memories from a newbie

Maybe the biggest lesson of the summer is that a good fix is rarely the first fix. This one went through several complete redesigns in review with the BDK maintainers and other contributors.

It started as a self-contained walk inside the wallet, then built on an earlier chain-layer effort ([#2235](https://github.com/bitcoindevkit/bdk/pull/2235)), and through discussion it ended up as a smaller, faster, memoized walk that only ever inspects your unconfirmed coins and their ancestors, so it doesn’t get slower as your wallet history grows.

A lot of the value came from other people poking holes in the approach. Performance concerns, edge cases like missing history, and questions about what “trust” should even mean. Contributing to open source is much less about writing the patch than about defending, breaking and rebuilding it in public.

## Where it stands

The trust redesign and the wallet delegation are open and under review, the confirmation primitive is open, and the `locked` category is open as a draft follow-up. The one hard dependency is release ordering: `bdk_wallet` builds against a published `bdk_chain`, so the wallet pieces are gated on the chain change shipping first. Next up are time-based timelocks and a future frozen/reserved category for coins the user locks manually.

## What else?

If you want to follow a little bit what I've been doing you can check all the path with those PRs and Issues:  

- [bitcoindevkit/bdk_wallet#431](https://github.com/bitcoindevkit/bdk_wallet/pull/431) was my first attempt, a walk directly in the wallet, later reworked to delegate to the chain walk once #2246 existed, and gained `min_confirmations`.
- [bitcoindevkit/bdk#2246](https://github.com/bitcoindevkit/bdk/pull/2246) manages the ancestry-based trust and eligibility in `bdk_chain`, which then sent me back to rework #431 on top of it.
- [bitcoindevkit/bdk#2263](https://github.com/bitcoindevkit/bdk/pull/2263) is a confirmation count primitive.
- [bitcoindevkit/bdk_wallet#538](https://github.com/bitcoindevkit/bdk_wallet/pull/538) is the `locked` balance category.

The issues that framed the work: [#16](https://github.com/bitcoindevkit/bdk_wallet/issues/16) and [#273](https://github.com/bitcoindevkit/bdk_wallet/issues/273) (the trust bugs), [#180](https://github.com/bitcoindevkit/bdk_wallet/issues/180) (locked coins).
