---
layout: post
title: "What I Learned Building Search and Reporting for Shopstr During Summer of Bitcoin"
date: 2026-06-26
author: Ayush Srivastava
categories: [Shopstr, Nostr, Stories]
image: ../assets/images/blog_content/NOSTR.png
---

When I started Summer of Bitcoin, I thought I was going to work on a clear technical project.

Add NIP 50 search. Add NIP 56 reporting. Connect them properly into Shopstr. Write tests. Get the pull requests reviewed. Move on.

That was the simple version in my head.

But once I actually got into the work, I realized the real project was not just about adding two protocol features. It was about understanding how a decentralized marketplace behaves when there is no single backend quietly holding everything together. It was about learning how relays behave in the real world, how users interact with protocol level data, and how small decisions around search, reporting, cache, and UI can shape trust inside a marketplace.

My project is called Search and Reporting for the Decentralized Bitcoin Marketplace, and I worked on it inside Shopstr. Shopstr is a Nostr based marketplace, which means listings, profiles, reports, and other marketplace activity are built around Nostr events and relays. That sounds clean when you describe it from far away, but the deeper you go, the more you realize that decentralized systems are not automatically simple. They just move the complexity to a different place.

In the first half of the program, the two main pieces of my project were merged.

The first was [NIP 50 marketplace search](https://github.com/shopstr-eng/shopstr/pull/502).

Before this work, marketplace discovery was mostly limited to client side filtering. That meant Shopstr could only filter through what it already had locally. With the NIP 50 search work, the marketplace can now ask search capable relays for matching listings. This gives Shopstr a relay backed search path instead of depending only on local filtering.

At first, that sounded like a straightforward feature. Send a search query to relays, get results back, show them in the marketplace.

But the real world version is more interesting.

Some relays are slow. Some relays return duplicate events. Some relays return incomplete results. Some relays do not support NIP 50 at all. So the work became less about simply adding a search request, and more about making search behave well when the network does not behave perfectly.

One of the most important pieces of feedback I got from Calva was around this exact issue. I had started from the Nostr docs and was building the search flow around the user selected relays. Calva pointed out that some relays do not actually support NIP 50, so Shopstr should have a default backup list for search when the user selected relays do not work. That feedback changed how I thought about the feature. A protocol implementation is not complete just because it follows the spec. It also has to survive the uneven reality of the network.

So the search work added fallback search relays, deduped listing results, and better handling for slow relays so one bad relay does not break the whole search experience. That was a small but important shift in my thinking. I started seeing Nostr integration not as one event going in and one event coming out, but as a full product flow that has to handle uncertainty.

The second major piece was [NIP 56 reporting](https://github.com/shopstr-eng/shopstr/pull/273).

Shopstr did not have a standard reporting flow for objectionable listings or seller profiles. In a marketplace, that is not a small missing piece. Discovery helps people find listings, but reporting helps the community respond when something should not be there.

The reporting work added a way to report both listings and seller profiles using NIP 56 kind 1984 report events. It touched the UI, the Nostr publishing path, the fetch flow, and the cache layer. Users can report from the product and profile flows, the report event gets published, and the event can also move through Shopstr's existing data flow.

This part taught me that moderation in decentralized applications is not the same as moderation in a centralized app. There is no single database table that decides everything. A report has to be a valid protocol event. It has to point to the right listing or profile. It has to be published through relays, cached properly, fetched back safely, and shown in a way that makes sense for the product.

That is where I started to understand the shape of Nostr components inside a real application. Events are not just data objects. They are the connection between the protocol and the user experience. If the event is wrong, the product behavior is wrong. If the fetch path is weak, the UI becomes unreliable. If the cache stores too much or exposes the wrong data, the trust boundary becomes unclear.

A lot of my learning came from that middle layer.

The part between "the protocol says this" and "the user sees this."

That middle layer is where most of the hard work lives.

I also worked on tests and CI, because after the search and reporting work started landing, it became clear that these flows needed stronger protection. A feature that works once locally is not enough. Search and reporting touch relays, cache, parser logic, UI behavior, and protocol event construction. If future changes break those paths silently, the feature is not really stable.

So I worked on closing the [test coverage gap](https://github.com/shopstr-eng/shopstr/pull/446).

That PR added GitHub Actions CI so the test suite and coverage checks can run automatically on pull requests. I also added and tightened follow up tests around NIP 50 search and NIP 56 reporting, covering cases like blank queries, normalized search text, fallback relays, duplicates, timeouts, report types, kind 1984 events, publish relays, and cache behavior.

The [search follow up tests](https://github.com/shopstr-eng/shopstr/pull/538) and [reporting follow up tests](https://github.com/shopstr-eng/shopstr/pull/527) made the project feel more complete. Not just "I added the feature," but "I helped make the feature safer to maintain."

One thing I have learned in open source is that code is only one part of the work. You also have to explain the change clearly. You have to respond to review. You have to understand why a maintainer is asking for a different approach. You have to write tests that prove the behavior, not just tests that make the coverage number look better.

That has been one of the most valuable parts of Summer of Bitcoin for me.

Before this program, I used to think of decentralization mostly as a big idea. User ownership. Open protocols. No central control. Those are important ideas, but working inside Shopstr made them much more practical for me.

Decentralization shows up in small engineering decisions.

It shows up when a relay does not support the thing you expect it to support.

It shows up when the same listing comes from two different relays and the app has to know it is the same thing.

It shows up when a report has to be a real protocol event, not just a button that sends data to a server.

It shows up when wallet data, cache behavior, storefront content, and user generated events all need clear trust boundaries.

That is what surprised me most about Bitcoin open source development. It is not just people talking about decentralization at a high level. It is very practical. Every line of code has to respect the fact that users are bringing their own keys, their own wallets, their own relays, and their own data into the product.

That makes the work harder, but also more meaningful.

The mentorship and review process also changed how I work. The Shopstr maintainers have been quick with feedback and open to discussion. When I opened PRs, whether for the core project or for security gaps, I got real review, not just approval for the sake of moving fast. That helped me understand the codebase better, but more importantly, it helped me understand the product better.

A merged PR is not the finish line by itself. It is a checkpoint.

The real question is whether the change still makes sense after review, after edge cases, after tests, and after thinking about how users will actually experience it.

At the midpoint of the program, the core NIP 50 search and NIP 56 reporting work is already merged. That feels good, but I do not see it as the project being "done." The second half is about making the work stronger. I want to keep testing relay failures, incomplete responses, fallback search behavior, duplicate events, cache edge cases, and reporting flows from publish to fetch to cache.

Because the goal is not just to say that search and reporting exist in Shopstr.

The goal is to leave them in a state that maintainers can trust.

Looking back, I came into Summer of Bitcoin thinking mostly about implementation. I wanted to ship meaningful code. I still care about that. But the biggest thing I am taking away is a deeper understanding of what it means to build in an open, decentralized system.

You are not just writing features.

You are building around uncertainty.

You are designing for users who hold their own keys.

You are working with networks that do not always behave the same way.

You are making trust visible in code.

And if you do the work properly, even a small feature like search or reporting becomes part of something larger. It becomes part of making decentralized marketplaces more usable, safer, and easier to trust.

That is what Summer of Bitcoin has given me so far. Not just a project, and not just a set of pull requests, but a clearer way to think about engineering itself.
