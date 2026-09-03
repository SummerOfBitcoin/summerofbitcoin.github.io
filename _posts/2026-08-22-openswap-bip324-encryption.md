---
layout: post
title: "Encrypting OpenSwap's Maker–Taker Transport with BIP324"
date: 2026-08-22
author: "Yogansh"
categories: [Development, Open-Source, Stories, Security]
image: assets/images/blog_content/2026-08-22-openswap-bip324-encryption.jpg
---

## Opening

For my Summer of Bitcoin project, I worked on **OpenSwap** (formerly Coinswap).
In a coinswap, a taker splits a payment across a route of makers, and each hop shuffles funds 
through a chain of contracts so that no single observer can link the sender's coins to the
receiver's. The privacy story depends on everything around the swap being as opaque as the swap itself.

The problem I set out to fix was the transport. Maker and taker exchanged every protocol message 
over **plaintext TCP** which was wrapped in a four-byte length-prefixed CBOR framing. Tor gave network-level
privacy, but the bytes between the two parties were readable by anyone controlling the exit node

My project was to put an encrypted transport over that message exchange, which needed to be 
authenticated by default.For this [rust-bitcoin/bip324](https://github.com/rust-bitcoin/bip324) was used 
which implements the [BIP324 spec](https://github.com/bitcoin/bips/blob/master/bip-0324.mediawiki).

## Technical background

During a swap a taker connects to N makers, fetches offers,
negotiates swap details, and then runs a scripted sequence of contract exchanges and signature round-trips.
OpenSwap has two protocol flows: an older **Legacy** flow and a newer **Taproot** flow, although they share the same message structure: `TakerToMakerMessage` / `MakerToTakerMessage`, serialized with CBOR.

Before my work, that structure was sent over a raw `TcpStream` with four big-endian bytes for the length, then the CBOR payload. Both sides
had their own copies of `read_message`/`send_message` implementation.
There was no encryption or message authentication.

BIP324 (Bitcoin's v2 transport) changes that. It gives us:

- an authenticated encryption handshake (a Noise-derived protocol) over the
  connection,
- AEAD encryption for every subsequent packet,
- a **session ID**, a shared secret established during the handshake
- and genuine vs decoy packet types, so traffic-analysis countermeasures which could be implemented later as needed

## The Journey

### First implementation

In the first substantial commit, I introduced a thin wrapper around the `bip324::Protocol` :

```rust
pub(crate) struct Bip324Stream {
    pub stream: Protocol<BufReader<TcpStream>, TcpStream>,
}
```

`new` cloned the socket and returns an encrypted stream which is used with `read_message`/`send_message`

The maker became the BIP324 **responder** and the taker the **initiator**, which mirrors how the 
protocol already behaves (taker connects, maker accepts). 

### The identity problem

BIP324's handshake is unauthenticated by default an active Man in The Middle can simply be the one
negotiating with both sides. 

The BIP324 spec anticipates exactly this. From
[bip-0324.mediawiki](bip-0324.mediawiki): v2 transport exports a *session ID*

The session ID is derived from a Diffie-Hellman negotiation is unique to each connection; no two connections ever share it.
If a signature is bound to a session ID, it cannot be replayed onto a different connection,
and an active man-in-the-middle who splits the link into two BIP324 connections forces two
different session IDs to exist (one with each honest peer). The maker signs the session ID it sees;
The taker verifies against the session ID *it* sees; those only match if they really are on the same encrypted
channel. Binding the identity to a per-connection session ID is what makes the
authentication catch an attacker.

The `rust-bitcoin/bip324` crate's high-level `Protocol`
didn't expose the `session_id()` accessor only lived on the low-level
`Handshake`, and I didn't want to wrap or reimplement that. Therefore, I contributed the accessor upstream:
[PR #167](https://github.com/rust-bitcoin/bip324/pull/167) ("Add session_id
accessors to Protocol and async Protocol") added a `SessionId` newtype and
`session_id()` accessors on both the sync and async `Protocol`, plus a round-trip
test asserting both peers derive the same session ID, closing
[issue #61](https://github.com/rust-bitcoin/bip324/issues/61). It was reviewed
and merged in June, and I bumped the pinned dependency to the rev that contains
it.

The approach I settled on binds the BIP324 session ID to the maker's existing **tweakable key*.
The maker uses this key to build the funding transaction. So a signature over the session ID 
from this key proves that the message is from the maker.

If only the entity controlling the maker's private key can produce a valid
signature over the session ID, then a verified channel provably terminates at
The maker we negotiated with.

#### When to authenticate
The interesting constraint was ordering. The identity to bind against (the tweakable point) is only known
after the offer arrives. The cost is that authentication happens *after* the handshake and offer fetch.
The signature is bound to the tweakable point, but the taker only learns the tweakable point from the offer
which arrives *after* the handshake. So authentication had to happen after
fetching the offer, not at the handshake boundary. I also hardened the binding on
both sides of the negotiation:

```rust
// The ack must commit to the exact tweakable point we authenticated against;
// anything else means the binding is compromised.
if tweakable_point != offer.tweakable_point {
    return Err(TakerError::General(format!(
        "AckSwap tweakable point mismatch: expected {}, got {}",
        offer.tweakable_point, tweakable_point,
    )));
}
```


### Stop reconnecting per phase

Once every phase required an authenticated handshake, the existing pattern of
opening a fresh connection every phase became wasteful. Negotiation,
contract exchange, finalization, and private-key handover each opened a new TCP
connection and repeated the handshake. That means re-verifying identity at every step.

I added connection caching to the swap state so the same connection is reused after the swap acknowledged message is received.

## Closing

Summer of Bitcoin gave me a real protocol to evaluate, and a codebase large enough that I was solving genuine
architecture problems. The full implementation is in [PR #876](https://github.com/citadel-foss/openswap/pull/876). 

---

