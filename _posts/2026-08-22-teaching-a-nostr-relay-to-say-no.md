---
layout: post
title: "Teaching a Nostr Relay to Say No: Building nostream's Access Layer"
date: 2026-08-22
author: "Anshuman"
categories: [ Nostr, Development, Open-Source, Stories, Security ]
image: assets/images/blog_content/2026-08-22-nostream-access-layer.jpg
---

I'm Anshuman. This summer I worked on [nostream](https://github.com/cameri/nostream), a production Nostr relay written in TypeScript, with Ricardo Cabral as my mentor. The project was an access engine: take a relay that is open by default and give operators a way to authenticate clients, protect events, and admit members. I also added NIP-98 so they can call the existing admin API with a signed event, without dropping live connections.

This is the final report.

## What I set out to do

Nostr is permissionless on purpose. Anyone can publish, anyone can subscribe, and a relay that wants to stay open should stay open. The problem is that some operators cannot run that way. They need authenticated sockets, events that only the author may publish, invite-only membership, and an admin API that is not just a shared password.

The brief was to build that layer *inside* nostream, not next to it:

1. Session state for [NIP-42](https://github.com/nostr-protocol/nips/blob/master/42.md) challenge-response AUTH on each WebSocket.
2. [NIP-70](https://github.com/nostr-protocol/nips/blob/master/70.md) protected events, rejected at ingest unless the socket is the author.
3. [NIP-43](https://github.com/nostr-protocol/nips/blob/master/43.md) access metadata and an automated state machine for membership requests.
4. [NIP-98](https://github.com/nostr-protocol/nips/blob/master/98.md) HTTP auth on the admin API, so operators can sign a request instead of only using a password.

## What I built

Four NIPs, one pipeline. Every WebSocket now has a `Nip42SessionManager`: one challenge per connection, multiple authenticated pubkeys, optional TTL, and replay protection so the same AUTH event cannot be reused to refresh expiry. Publish checks, NIP-70, restricted reads, and NIP-43 join/leave all read that session when those features are on. Restricted reads apply to REQ, live broadcasts, and COUNT.

Protected events (`-` tag) are dropped unless the publisher has AUTHed as the author. Nested reposts that embed a protected event are blocked too.

Membership is invite-based. Operators mint a claim code (`nostream invite create`). A client AUTHs, publishes a kind 28934 join with that claim, and is admitted. Kind 28936 leave revokes admission and requires the NIP-70 tag. When `nip43.enabled` is on, writes are members-only even if Lightning payments are off, and NIP-11 only advertises 43 in that case.

On HTTP, kind 27235 `Authorization: Nostr …` is verified (id, Schnorr signature, `u` / `method` / `payload`, clock skew). Redis `SET NX` makes each auth event one-time. Express middleware on `/admin` accepts it next to password sessions, fail-closed on an empty pubkey allowlist.

## Major technical/design decisions

**Fail closed, off by default.** `nip42.authRequired`, `nip43.enabled`, and `admin.nip98.enabled` are all false until an operator turns them on. An empty NIP-98 allowlist authenticates nobody. Existing public relays do not change behavior on upgrade.

**NIP-11 `restricted_writes`, not `auth_required`.** `auth_required` means AUTH before *any* action on the connection. We only gate publishes (and optionally some reads). Advertising the wrong flag would break clients that treat the document as gospel. The setting is `nip42.authRequired` internally and `limitation.restricted_writes` on the wire.

**Small PRs, in order.** Types and schemas first, then AUTH, then NIP-70, then invite storage, then join/leave, then HTTP verifier, then admin middleware, then the session object that the earlier pieces deserved. Kind 28935 (client asks the relay for an invite) waited until membership already worked, because that path puts a secret claim onto a live subscription.

**Secrets never hit the fanout.** Join requests are not broadcast: the `claim` tag is a bearer token. Kind 28935 invite-on-request is designed the same way (ephemeral, unstored, only on the asking subscription) and is still in [PR #738](https://github.com/cameri/nostream/pull/738).

**Follow the current NIP, not the proposal wording.** The original brief mentioned kind 843. Current NIP-43 uses 28934 / 28935 / 28936 for the client request flow. We implemented join, leave, and operator invite minting against those kinds, not the whole NIP.

**NIP-98 beside passwords, not instead of them.** Another contributor opened a PR to replace password login with NIP-98. The version that merged keeps password sessions and adds signed HTTP auth as a second door, so a relay can cut over without locking the operator out.

## What shipped

All of this is in [cameri/nostream](https://github.com/cameri/nostream) `main` unless noted.

**Phase 1 (before midterm, 29 June – 3 July)**

- [PR #622](https://github.com/cameri/nostream/pull/622) — NIP-42 types, schemas, constants
- [PR #629](https://github.com/cameri/nostream/pull/629) — AUTH handler and WebSocket wiring
- [PR #643](https://github.com/cameri/nostream/pull/643) / [#644](https://github.com/cameri/nostream/pull/644) — NIP-70 detection and ingest rejection
- [PR #650](https://github.com/cameri/nostream/pull/650) — invite codes, atomic `claimCode`, migration

**Phase 2**

- [PR #702](https://github.com/cameri/nostream/pull/702) — restricted reads of encrypted kinds (REQ, live, COUNT)
- [PR #676](https://github.com/cameri/nostream/pull/676) — join / leave strategies, admission cache, NIP-11 advertising
- [PR #716](https://github.com/cameri/nostream/pull/716) — session manager, optional TTL, `authRequired`
- [PR #732](https://github.com/cameri/nostream/pull/732) — `nostream invite create`
- [PR #722](https://github.com/cameri/nostream/pull/722) / [#725](https://github.com/cameri/nostream/pull/725) / [#726](https://github.com/cameri/nostream/pull/726) / [#730](https://github.com/cameri/nostream/pull/730) — NIP-98 verifier, absolute URL from `relay_url`, Redis one-time claims, admin middleware

Settings and operator docs landed in `CONFIGURATION.md`, `CLI.md`, and the README.

## What remains

[PR #738](https://github.com/cameri/nostream/pull/738) is open: kind 28935 invite-on-request. An authenticated client `REQ`s an invite; the relay mints a code and returns a relay-signed ephemeral event on that subscription only. It is implemented and waiting on review. It shipped last on purpose.

Relay-published membership-list / add-remove events (kinds 13534 / 8000 / 8001) have constants and no handlers. Role events (kind 33534) are not implemented. Join requests also do not yet require the NIP-70 `-` tag that the current spec wants. What shipped is invite admission (admitted or not), not the full NIP-43 roster and roles model.

## What I learned

A NIP is a contract with clients you do not control. Getting `restricted_writes` vs `auth_required` wrong is not a docs nit; it is a compatibility bug.

Replay is the boring bug that eats auth systems. The same signed AUTH event refreshing a TTL, or the same NIP-98 event hitting POST twice, is enough to undo the cryptography. Connection-scoped AUTH ids and Redis `SET NX` exist because of that.

Maintainer review is part of the design. Restricted reads are "author or `p`-tagged recipient," not "author only," because that is how DMs work. Extracting a session object late was better than pretending the first AUTH handler was finished.

And the protocol moves. Kind 843 in a proposal is not a reason to ignore 28934 in the spec.

## What happens next

Merge [#738](https://github.com/cameri/nostream/pull/738) if the review holds. After that, membership-list events are the obvious follow-up for relays that want a public roster, not only a private admit bit.

If you run nostream and want a closed relay: turn on `nip43.enabled`, mint a code, and keep NIP-98 off until `allowedPubkeys` is set. If you want AUTH on publishes without membership, `nip42.authRequired` is enough.

Thanks to Ricardo for the reviews and the sequencing, and to the other nostream SoB contributors who let me read and comment on their PRs while this was going in.

## Links

- Relay: [github.com/cameri/nostream](https://github.com/cameri/nostream)
- Session manager (best single PR): [#716](https://github.com/cameri/nostream/pull/716)
- Join / leave: [#676](https://github.com/cameri/nostream/pull/676)
- NIP-98 on admin: [#730](https://github.com/cameri/nostream/pull/730)
- Invite-on-request (open): [#738](https://github.com/cameri/nostream/pull/738)
