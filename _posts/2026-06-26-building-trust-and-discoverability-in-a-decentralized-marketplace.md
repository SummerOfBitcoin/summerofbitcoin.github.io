---
layout: post
title: "Search and Reporting for the Decentralized Bitcoin Marketplace"
date: 2026-06-26
author: Ayush Srivastava
categories: [Shopstr, Nostr, Stories]
image: ../assets/images/blog_content/NOSTR.png
---

I'm Ayush Srivastava, a Computer Science student at the University of Colorado Boulder, and this summer I have been working as a Summer of Bitcoin contributor, Batch of 2026.

You can find me on [LinkedIn](https://www.linkedin.com/in/ayushshrivastv/) and [GitHub](https://github.com/ayushshrivastv).

This summer has not just been about writing code. It has been about learning what it really means to build in the open, where every change is reviewed, questioned, tested, and shaped by people who care deeply about the systems they are maintaining. My work has been with Shopstr, a decentralized Bitcoin marketplace built on Nostr, where I have been focused on making the marketplace easier to discover and safer to use through NIP 50 search and NIP 56 reporting.

When I first looked at the project, it seemed simple on the surface. Add search. Add reporting. Connect the flows. Get the pull requests merged. But the deeper I went, the more I understood that a decentralized marketplace is not held together by one server or one database. It is held together by relays, events, cache, wallets, signatures, user trust, and the small engineering decisions that decide whether the product actually works when the network is messy.

That is what made this summer meaningful for me. I was not just adding features to Shopstr. I was learning how trust is built in a system that does not have a center.

The project I worked on is called Search and Reporting for the Decentralized Bitcoin Marketplace. The goal was to improve two important parts of Shopstr. First, marketplace discovery, so users can find listings more reliably. Second, reporting, so users can report objectionable listings or seller profiles through a proper protocol based flow.

The first major part of my work was NIP 50 marketplace search.

[Search PR](https://github.com/shopstr-eng/shopstr/pull/502)

Before this work, Shopstr's marketplace search was mainly based on client side filtering. That meant the app could only filter through listing data it already had locally. It worked, but it limited discovery. A marketplace needs search to feel alive. If users cannot find what they are looking for, the marketplace becomes smaller than it really is.

With the NIP 50 search implementation, Shopstr now has a relay backed search path. Instead of only filtering locally, the app can ask search capable relays for matching marketplace listings. That sounds simple when you write it in one sentence, but the actual work was more layered.

In a centralized app, search usually means sending a query to one backend and trusting that backend to return results. In a Nostr based marketplace, search has to deal with relays. Some relays are fast. Some are slow. Some return duplicate events. Some return incomplete results. Some do not support NIP 50 at all.

That was one of the most important things I learned from this project. A protocol feature is not complete just because it follows the spec. It has to work with the reality of the network.

One of the best pieces of feedback I received came from Calva during the NIP 50 search review. I had started from the Nostr docs and was building the search flow around the user selected relays. Calva pointed out that some relays do not actually support NIP 50, so Shopstr should have a default backup list for search if the user selected relays do not work. He also suggested search relays like relay.noswhere.com, search.nos.today, antiprimal.net, and relay.ditto.pub.

That feedback changed how I thought about the feature. I stopped thinking of search as only "send a query and show results." I started thinking of it as a reliability problem. What happens when a relay does not support search? What happens when one relay is slow? What happens when the same listing comes back from more than one place? What should the user see when the network is not clean?

So the search work included fallback search relays, deduped listing results, and better handling around slow relay behavior. The goal was not just to make search exist. The goal was to make search feel dependable in a decentralized environment.

The second major part of my work was NIP 56 reporting.

[Reporting PR](https://github.com/shopstr-eng/shopstr/pull/273)

Shopstr did not have a standardized reporting flow for objectionable listings or seller profiles. In a marketplace, that matters. Discovery helps users find listings, but reporting helps the community respond when something should not be there.

The NIP 56 reporting work added a way to report both listings and seller profiles using kind 1984 report events. It touched the user interface, the publishing flow, the fetch flow, and the cache layer. Users can report from marketplace and product flows, the report gets created as a proper Nostr event, and the event can move through Shopstr's existing data flow.

This was where I started to understand the deeper shape of a decentralized application. A report is not just a form submission. It is an event. It needs to point to the right listing or profile. It needs to be built in the correct format. It needs to be published, cached, fetched back, and handled safely.

In a centralized marketplace, reporting can be hidden behind a private database and admin system. In a decentralized marketplace, the reporting flow has to respect the protocol. It has to fit into the network. It has to be understandable to the product, but also valid in the wider Nostr model.

That middle layer became the most interesting part for me.

The space between "the protocol says this" and "the user experiences this."

That is where the real engineering lives.

Working on NIP 50 and NIP 56 taught me that Nostr components are not just separate pieces of code. They are part of one flow. Events are created, signed, published through relays, fetched back, parsed, deduplicated, cached, and shown in the UI. If one part is weak, the whole experience becomes unreliable.

That is why testing became such an important part of the project.

After the main search and reporting work landed, I worked on improving the test and CI story as well.

[CI and test coverage PR](https://github.com/shopstr-eng/shopstr/pull/446)

This added GitHub Actions CI so the test suite and coverage checks can run automatically on pull requests. I am proud of this work because it helps the project beyond my own feature. It gives maintainers a stronger safety net. It means future changes can be checked more consistently before they land.

I also worked on follow up tests for search and reporting.

[NIP 50 search test coverage](https://github.com/shopstr-eng/shopstr/pull/538)

[NIP 56 reporting test coverage](https://github.com/shopstr-eng/shopstr/pull/527)

The search tests cover things like blank queries, normalized search text, fallback relays, duplicate results, timeout failures, cache failures, and clearing temporary UI results. The reporting tests cover report types, kind 1984 event creation, publish relays, and cache behavior.

Those tests mattered because they turned the work from "this feature works when I try it" into "this behavior is protected." That is a big difference in open source. A feature is only as strong as the confidence maintainers have in changing it later.

One of the biggest shifts for me during Summer of Bitcoin has been understanding open source review as part of the engineering process, not something separate from it. Before this, I used to think mostly in terms of writing the code and getting it merged. Now I see review differently. Review is where the design becomes sharper. It is where assumptions get exposed. It is where maintainers bring years of context that you cannot get just by reading the code.

The Shopstr maintainers have been quick with feedback and open to discussion. That made the work move faster, but more importantly, it made the work better. A comment on a PR was often not just a requested change. It was a lesson in how the product should behave.

That is what happened with fallback search relays. That is also what happened with test coverage and edge cases. The feedback pushed the work from "technically implemented" to "more reliable in practice."

And that, to me, is the real heart of Bitcoin open source development.

It is practical.

It is not just people talking about decentralization at a high level. It is people asking very direct questions. What happens if this relay fails? What data are we trusting? What should be cached? What should never be stored in browser localStorage? What happens if a seller controls this field? What breaks when the user refreshes? What does the code protect, and what does it assume?

Those questions are not abstract. They show up in the code every day.

Working on Shopstr made decentralization feel real to me. Not as a slogan, but as a set of engineering constraints. Users bring their own keys. Relays may behave differently. Data can come from many places. Wallet flows need careful trust boundaries. Marketplace safety cannot depend only on one central authority. Every feature has to survive in that environment.

That makes the work harder. But it also makes the work more meaningful.

At the midpoint of the program, the core NIP 50 search and NIP 56 reporting implementations are already merged into Shopstr. That is a good feeling, but I do not see the project as finished just because the main PRs are merged. The second half is about proving that these flows hold up under real conditions.

For search, that means testing relay failures, incomplete relay responses, fallback search relays, duplicate listings, cache behavior, blank queries, and slow responses.

For reporting, that means testing the full flow from creating the kind 1984 event to publishing, caching, fetching, and handling both listing reports and seller profile reports correctly.

The goal is not only to say that Shopstr now has search and reporting. The goal is to leave behind search and reporting that maintainers can trust after the program ends.

Looking back, I came into Summer of Bitcoin wanting to ship meaningful code. I still care about that. But the most valuable thing I have gained is a clearer way of thinking about engineering.

Good engineering is not just building the happy path. It is understanding what happens when the world is imperfect.

Relays fail. Data duplicates. Users refresh. Caches go stale. Protocol support is uneven. Security assumptions get tested. And the job of the engineer is to build anyway, with enough care that the product still feels stable to the person using it.

That is what I learned this summer.

I learned that open source is not just about writing code in public. It is about thinking in public, improving through review, and building something that can outlast your own contribution.

I learned that decentralization is not just a philosophy. It is a responsibility that shows up in small decisions.

And I learned that even a feature like search or reporting can carry something bigger inside it. It can carry trust. It can carry safety. It can make a marketplace easier to use and easier to believe in.

That is what Summer of Bitcoin has given me so far. Not just a project. Not just pull requests. But a deeper understanding of what it means to build systems that do not rely on a center, and still have to work for real people.
