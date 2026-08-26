---
layout: post
title: "Implementing Read Permissions in strfry"
date: 2026-08-16
author: Anurag Kumar Mishra # Must match the name in _config.yml
categories: [Nostr, Stories]  # See available categories below
image: ../assets/images/blog_content/strfry.svg
---

## What the heck is even strfry?
In short, a nostr relay. A fast one. It's in fact one of the fastest nostr relays that there is (no I don't mean you can't write slow and sluggish software in C++) (no I don't mean you can't write slow and sluggish software in C++). It's written in C++ which compiles to native code. So it's super fast and efficient (no I don't mean you can't write slow and sluggish software in C++). It uses minimal dependencies and does not require configuration to get you started running a nostr relay. Not even configuring a database (what?). Yeah, it uses LMDB which does not require any prior configuration (cries in Postgres). You can get it running in like 5 minutes if you are not using hardware from the 1960s. For more information, check [the strfry github repo](https://github.com/hoytech/strfry).

## Authentication, permissions and all that
The Nostr protocol is pretty open. You send an event, relays relay it and it's on the internet for everyone to view. Okay that's pretty wild, but what if someone wants to make sure that if X is the author of an event, only X can publish the event? Every event has an `pubkey` field that is the pubkey of the person who *signs* the event (notice, not who *publishes* the event). In some cases, X would want that their event is not spread to the entire world and is scoped to a particular community. The protocol defines a way to make events **protected**. When a protected event is published, the relay may check that client is authenticated and is the author of the event. So that's for the write-side permissions.

## DMs
Nostr also supports DMs. Like any other, DMs are also normal events but differ in that they are kind 4. The content is encrypted such that only the receipient can decrypt. The problem here is, that if there is no restriction on what events clients can fetch, they can at least read the DM metadata like the author and the receiver. This is at least not desired. So we need some kind of read-side restrictions as well. This is the work I have been doing this summer.

## Read restrictions
strfry now has two config options:
1. `restrictedReadKinds`: defaults to `"4,1059"`
2. `restrictReadToInvolvedPubkey`: defaults to true
They are fairly self explanatory but here are some details.
When `restrictedReadKinds` is set for a kind, events of that kind can only be read by authenticated clients. There were some performance decisions to be made but we agreed on filtering the events post-query.
When `restrictReadToInvolvedPubkey` is set to true, those restricted events can only be read by the authenticated client if that client's pubkey is involved in the event some way or the other. They could be the author or one of the tagged receipients.
And that's it. No metadata leak!

## Final words
There was a fair amount of thinking involved because we didn't want any degradation in performance due to event filtering. strfry is multithreaded, there are workers processing requests, workers monitoring db changes, etc. The auth information had to be shared among these threads. We chose that threads that require the information maintain there own copy (updated via messages) instead of having any kind of shared data.
After the `AUTH` and read-gating work was done, I have spent a fair amount of time trying to further optimise the relay performance. For example, the relay would flush out every event as it processed. I have made some changes so that now we can batch events before dispatching to the websocket thread, which means significantly lesser mutex locks and unlocks and overall better performance.
I also wrote a proper test suite that makes writing tests, both unit and integration, much easier. The tests have been wired up to run in the CI which means we have one extra layer of verification before a pull request is merged.

For me personally, this is the kind of project I have always wanted to work on. It has the right amount of relatively lower-level work.

## Work Artifacts
- A short demonstration of the read restrictions in action is available here: https://youtu.be/3RWaxU3Dkig
- PRs: 
  - https://github.com/hoytech/strfry/pull/239
  - https://github.com/hoytech/strfry/pull/250
  - https://github.com/hoytech/strfry/pull/257
  - https://github.com/hoytech/strfry/pull/258
  - https://github.com/hoytech/strfry/pull/261
