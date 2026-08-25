---
layout: post
title: "What Shopstr Taught Me About Real Open Source"
date: 2026-08-18
author: Ayush Srivastava
categories: [Shopstr, Nostr, Security, Open-Source, Stories]
image: ../assets/images/blog_content/NOSTR.png
---

<figure>
<img src="../assets/images/blog_content/NOSTR.png" alt="Nostr logo artwork"/>
<figcaption>Nostr was the protocol layer behind the search, reporting, relay, identity, and event flows I worked with this summer.</figcaption>
</figure>

I will start this by making one thing clear: before this summer, I knew working on a real open source project would be different from building something on my own, but I do not think I fully understood how different.

When you build your own project, you know where everything is because you were the person who put it there. If something breaks, there is a decent chance you also remember the questionable decision you made three days ago that caused it.

Working on [Shopstr](https://shopstr.store/) was nothing like that. I was joining an existing marketplace with its own architecture, users, Nostr relay flows, caching, wallet code, checkout code, old decisions, new decisions, and probably a few decisions nobody remembered making anymore.

My Summer of Bitcoin project was called "Search and Reporting for the Decentralized Bitcoin Marketplace," which sounded very contained when I first read it. I was supposed to add NIP-50 search and NIP-56 reporting to Shopstr. Search and reporting. Two things. I could explain the entire project in one sentence.

By the end of the summer, my GitHub PR list included search, reporting, CI, test coverage, storefront security, API authorization, Cashu proofs, wallet secrets, checkout validation, marketplace bugs, MCP fixes, NIP-58 badges, and enough time spent thinking about localStorage that I would be perfectly happy not hearing that word for a while.

## Search That Has To Work On A Real Network

The first big part of the project was NIP-50 marketplace search. Shopstr already had marketplace browsing, but search depended a lot on filtering what was already available locally. NIP-50 lets clients ask search-capable Nostr relays for matching events, so the goal was to make marketplace search actually go out and find listings instead of only looking through what Shopstr already had.

I started from the Nostr docs and built the search flow around the relays selected by the user. At first this made complete sense to me. The user has relays, NIP-50 defines search, so ask those relays to search. Done, right?

Then during review my mentor, Calva, pointed out that some relays simply do not support NIP-50. This sounds painfully obvious when I write it now, but at that point I had been thinking mostly about whether my implementation followed the protocol correctly, not whether every relay in the real world was going to cooperate with it.

His suggestion was to keep a known set of search relays as backups, so if the user's relays could not handle the search, Shopstr would still have somewhere useful to ask. I added fallback relays, handled duplicate results coming back from different places, and made sure one slow relay could not make the whole search feel broken.

That was probably one of the first moments this summer where something clicked for me. Reading a protocol and implementing it correctly is one part of the job. Making that protocol work inside a real product, on a real network, where other software does not always behave exactly how you hoped, is another part entirely.

## Reporting Is A Product Feature, Not Just A Button

The other main feature was NIP-56 reporting. Shopstr did not have a standard Nostr reporting flow for objectionable listings or seller profiles, so I worked on adding one that fit into the existing application instead of building some completely separate moderation system beside it.

Reports had to be created correctly, published through the relay flow, fetched again, cached, and connected to the UI where users would actually report a listing or seller. I liked working on this because it was one of those features that does not sound especially exciting when you describe it, but it matters once you think about the kind of product Shopstr is.

An open marketplace is great because people can publish and sell without one company controlling everything, but "open" does not somehow mean nobody will ever post anything abusive, misleading, or unwanted. Someone eventually needs a report button. Nobody is going to open Shopstr for the first time and say, "Wow, incredible NIP-56 integration," but they might care a lot about it on the day they actually need it.

I also added stronger tests around the report events, report types, relay publishing, and cache behavior because by this point I was already starting to understand that getting a feature working once is not really the end of the work. You want to be reasonably sure the next person who changes something nearby does not accidentally break it six weeks later.

## Tests Became A Way To Understand The Codebase

Testing became a much bigger part of my summer than I expected. At one point I worked on adding a GitHub Actions CI workflow that runs the Jest test suite and checks coverage on pull requests. I also spent time tightening tests around NIP-50 search, NIP-56 reports, NIP-99 listing parsing, caching, Nostr order messaging, Cashu wallet behavior, and other areas that felt risky enough that "I clicked around locally and it seemed fine" was not a great long-term plan.

I used to think about tests mostly as something you write because you are supposed to write tests. This summer made them feel much more practical. In a codebase you did not create, a good test tells you something about what the project considers important. It is almost like someone left you a note saying, "This behavior looks small, but please do not mess with it."

When I was changing search behavior across several relays, or touching wallet state, or adjusting authorization logic, having tests around the weird cases made a huge difference. Blank queries, duplicate listings, relay timeouts, failed caches, old stored values, unexpected status names. These are not the fun happy-path screenshots you put in a PR description, but they are usually the things waiting to embarrass you later.

## The Bugs Outside The Original Project

Once I had spent enough time inside Shopstr, I also started noticing issues that had nothing to do with my original project. Some were normal bugs. One marketplace bug, for example, could show an image from the previous listing after you opened a different product because old selected image state was still hanging around. The title changed, the listing changed, but somehow the image was still living in the past.

I traced that down and fixed it, and I remember liking that bug more than I probably should have because the whole problem could basically be described as: "This state is old. It should not be old."

Other issues were more serious. I found places where seller-controlled storefront content could become unsafe JavaScript links, policy HTML could be rendered in a dangerous way, cached messages could be read without enough proof of ownership, failed relay publish queues could be accessed too freely, and shared cached events could be deleted without the kind of authorization checks you would want on a destructive endpoint.

A lot of these problems had the same shape. The frontend sends some value to the backend, and somewhere along the way the system trusts that value more than it should. It is one thing to learn "never trust the client" as a general security rule. It is another thing to stare at a real request path and realize, oh, we are literally trusting the client here.

## Wallet State Made Security Feel Concrete

The checkout and wallet-related issues probably changed how I think about security the most, mostly because the consequences were easier to picture. I worked on revalidating stored discount values before checkout because browser storage is editable and should not get the final say in how much somebody pays. I also worked on moving listing mint quote validation to the server so the backend could fetch the listing and work out the real amount instead of trusting a price handed over by the browser.

When money is involved, "the frontend already checked it" stops sounding reassuring very quickly.

Then I got into Cashu proofs and wallet credentials. Shopstr had flows where spendable Cashu proofs could be persisted in localStorage, and there were similar concerns around Nostr Wallet Connect connection strings and NIP-46 signer information. These are not harmless preferences like whether somebody selected dark mode. They are secrets or, in the case of Cashu proofs, potentially spendable value. So I worked on keeping sensitive information in runtime memory instead of leaving it sitting in persistent browser storage.

This was also where I learned that security changes tend to arrive with a lot of relatives. You change where one piece of wallet state lives, and suddenly you are thinking about app hydration, recovery, old data from previous versions, components that expect the value to exist, tests that depend on the previous flow, and what happens when the page refreshes.

## Small Diffs Can Matter A Lot

One of the things I enjoyed most about the summer was that I was not boxed into only doing what was written in the original project description. If I found something useful and could understand the problem well enough, I could work on it.

That is how I ended up fixing MCP order authorization checks, a custom-domain ownership issue, discount code API problems, storefront mutation routes, OpenGraph preview URL validation, product image privacy hardening, and a completed-order status mismatch.

Some of these PRs were tiny compared with the main NIP features. Some were a couple of lines in exactly the right place. But I stopped measuring the value of the work by how big the diff looked. A ten-line ownership check can matter more than a giant feature if the missing ten lines let one user change another user's data. A one-line state reset can matter if every person browsing listings can see the bug.

That was another thing this summer changed for me. I care a lot less now about whether a change looks impressive in isolation and a lot more about whether it fixes something real.

## Badges Pulled The Threads Together

Toward the end of the summer I also worked on NIP-58 profile badges, which was fun because it brought together a lot of things I had slowly become comfortable with.

At first glance, badges sound almost laughably simple. A user has a badge, so show the badge. Except Nostr does not hand you one neat little "badge object" and wish you a nice day. The profile badge list points you somewhere, the badge award is another event, the badge definition is another piece, and you still need to make sure the award actually belongs to the profile you are about to decorate with it.

So I worked through fetching the events, resolving the chain, validating it, rendering the badge in the UI, and adding tests around the whole flow. A few months earlier that kind of feature would have felt like a bunch of unrelated systems I needed to figure out separately. By then it felt much more normal.

Fetch the right events. Understand what can be trusted. Validate the relationship. Fit it into the existing flow. Test the weird parts. Make sure the UI does not lie. That change in how comfortable I felt inside the codebase was probably a better sign of progress than any individual PR.

## Review Was Where The Work Got Sharper

My mentor also made the whole process easier because communication never felt complicated. If I was unsure about something, I could ask. If there was an issue with an approach, the feedback usually came with the reason behind it, which I appreciated much more than somebody simply saying "change this."

The NIP-50 relay feedback is a good example because the useful part was not just getting a list of fallback relays. It was understanding why relying only on the user's relays was too optimistic in practice.

At the same time, I had enough room to go find problems, read through the code, try things, get something wrong, fix it, and actually own the work. I never felt like I was just implementing a checklist somebody had already solved in their head. A lot of my learning happened in that slightly uncomfortable middle part where I understood enough to start, not enough to know every answer, and had to keep digging until the feature or fix made sense.

## What I Am Taking From This Summer

The most valuable part of Summer of Bitcoin for me was getting to experience what it feels like when the code actually matters.

In a personal project, if I break something, maybe I sigh, fix it tomorrow, and nobody notices. In a real project, somebody may be using the thing you are changing. Another contributor might build on it later. A bad authorization assumption can expose someone else's data. A checkout mistake can involve real money. A relay that does not support the protocol you expected can make a perfectly reasonable implementation feel broken to the user.

Your code also has to survive review, tests, old behavior, edge cases, and other people reading it months later without having access to whatever explanation was living in your head when you wrote it. That sounds obvious, but actually spending a summer inside that process made it feel very different from knowing it in theory.

So yes, technically, my Summer of Bitcoin project was about NIP-50 search and NIP-56 reporting for Shopstr, and those were still the center of what I came there to build. But when I look back at the summer, I do not really think of it as "the summer I implemented two Nostr NIPs."

I think of it as the summer I got much more comfortable walking into a codebase I did not create, asking why something works the way it does, following a problem across frontend, backend, relays, caches, tests, and wallet state, and occasionally finding something completely unrelated along the way that was worth fixing anyway.

I learned a lot about Nostr and Bitcoin applications, obviously, but I also learned that real software is much messier than the neat project description you start with. There are always edge cases, old assumptions, awkward state, unreliable networks, surprising security boundaries, and that one test that fails only after you were already convinced you were done.

And weirdly, that messy part ended up being the part I liked most.

If you want to understand the project beyond the pull requests, the best way is to try the product itself. Open [Shopstr](https://shopstr.store/), search for something in the marketplace, click through a few listings, and notice how much of the experience is being held together by Nostr events, relays, wallet state, and small trust decisions that are easy to miss when everything works.

That is the point of the work I did this summer. Search should help people find real listings without needing to understand which relays support which protocol. Reporting should be there when the marketplace needs a way to surface bad listings or profiles. Wallet and checkout paths should be careful with user funds even when the browser is not a trustworthy place to store secrets. The product should feel usable without hiding the fact that decentralization comes with different responsibilities.

So try Shopstr, break your assumptions a little, and if you find something that can be better, open an issue or contribute a fix. That is how this summer worked for me: one real product, one real codebase, and a lot of small problems that turned out to be worth caring about.

## Links To The Work

- [NIP-50 search PR](https://github.com/shopstr-eng/shopstr/pull/502)
- [NIP-56 reporting PR](https://github.com/shopstr-eng/shopstr/pull/273)
- [CI and test coverage PR](https://github.com/shopstr-eng/shopstr/pull/446)
- [NIP-50 search test coverage](https://github.com/shopstr-eng/shopstr/pull/538)
- [NIP-56 reporting test coverage](https://github.com/shopstr-eng/shopstr/pull/527)
