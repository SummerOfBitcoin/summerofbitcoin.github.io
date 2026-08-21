---
layout: post
title: "Building Shopstr MCP: My Summer of Bitcoin Journey"
date: 2026-08-20
author: Aryan Ashok Jain
categories: [Shopstr, Nostr, AI, Stories, Open-Source]
---

When I started my Summer of Bitcoin project, I thought I would mainly be building an MCP server around Shopstr. What I ended up working on was much broader-search, pagination, relay fetching, product parsing, reviews, follow/unfollow, caching, and a lot of debugging around edge cases that only appeared once the features were actually being used

Shopstr is a Nostr-based marketplace where products, sellers, and reviews are published as public events on relays. My project was to build a **read-only MCP server** that would allow AI agents to query this data through tools such as product search, product details, seller information, reviews, and categories.

## Building for an AI Agent

One thing I quickly realized was that designing for an AI agent is different from designing for a human user. The amount of information returned matters because it directly affects the model's context.

This led to ideas such as **response budgeting**, where responses are capped to avoid unnecessarily consuming context, and adding metadata and hints to help an agent understand what it can do next instead of guessing. Pagination also became important because returning everything at once is not practical for an agent.

## Learning Pagination by Extending It

The original cursor-based pagination was implemented by my mentor Gautam.

Before I understood it properly, my instinct was to add a simple timestamp cursor and move on. But once I went through the existing implementation, I understood that the `seen` state was protecting against more than just straightforward duplicates.

For example, if a product had multiple versions of its event, simply using a timestamp boundary could allow an older version of the same product to appear on a later page. From the user's perspective, that would look like two different listings even though it was the same product.

There were also trade-offs around how many events should be stored in the cursor. Keeping every scanned event could make the cursor grow very quickly, especially when a search returned a large number of non-matching events. So part of the work involved thinking about which events actually needed to remain in `seen` to preserve correctness without unnecessarily growing the cursor.

This was one of the better learning experiences of the project because I wasn't writing the pagination system from scratch. I had to understand someone else's design first and then extend it without breaking the guarantees it was already providing.

## Search and Pagination

I worked on integrating **NIP-50 search** with the existing product search. The first approach seemed simple: fetch normal relay results and NIP-50 results in parallel and merge them.

The problem appeared when pagination was added. NIP-50 results are relevance-ranked rather than naturally ordered by time, so using the same timestamp cursor did not make sense. Re-fetching them on the next page could return the same results, which would then be removed as duplicates.

Working through this made me understand that pagination is not just about adding an `until` parameter. The cursor has to preserve enough state to make sure an agent can move through results without repeatedly seeing the same products.

## When the Library Already Has the Answer

One of the more interesting debugging problems happened outside the MCP server itself. The Shopstr marketplace could take two to three minutes to start loading when one of its configured Nostr relays was slow or unreachable.

I first traced the issue and realized that our code was effectively connecting to all relays first and only then starting the subscription. Logically, each relay's connection and data fetching should be independent .A slow relay should not prevent a healthy relay from returning data.

I initially thought I would need to change how relay connections were handled. Instead, I went into the actual `nostr-tools` source and found that `SimplePool` already connects and subscribes to relays independently and in parallel. The problem was in our own `NostrManager` wrapper, which was blocking that behavior.

Fixing the wrapper, along with the stale connection promise and missing connection timeout, removed the unnecessary two-to-three-minute stall.

That was one of the biggest lessons for me: **before implementing something around a library, understand what the library already does. Sometimes the functionality you need is already there, and the problem is simply how you are using it.**

## Building Follow/Unfollow from Scratch

Follow/unfollow was different because there was no existing implementation to extend. I built it end-to-end using **NIP-02 contact-list events**.

The basic operation was simple-add or remove a `["p", pubkey]` tag and publish a new kind:3 event. The difficult part was everything around it.

Testing repeated follow/unfollow actions exposed several subtle issues: 
- rapid actions could race and build off the same stale contact list
- background refreshes could overwrite newer local state with stale data, and 
- an unresponsive relay could be misread as an empty contact list-risking a real follow list being silently wiped. 

I fixed these with serialized mutations, monotonic timestamps, and a hard constraint that an empty contact list is never trusted unless it is actually confirmed; otherwise, the action is blocked and the user is notified. Along the way, I also traced a subtle timestamp bug back to the Postgres driver returning created_at as a string, causing a numeric increment to become string concatenation.

This was a good reminder that implementing the feature itself is often the easy part. Making the feature reliable requires thinking through what can happen when multiple things are happening at the same time.

## One Parser, One Source of Truth

Another issue was that the main Shopstr application, root MCP server and the standalone MCP package had separate product parsers. Over time, they could interpret the same Nostr event differently.

I discussed different ways of sharing the parser including creating a separate shared package or maintaining a duplicated file and we eventually went with a simple approach: maintain one canonical parser and copy it into the MCP package during the build process. This keeps the two implementations aligned without introducing another package just for a small shared module.

I also worked on adapting product and review parsing around the GammaMarkets specification, including the way review events reference products and sellers.

This taught me that it's better to proceed with simple solution and avoid unnecessary complexity that can break stuff.

## What Shipped

By the end of the program, the main MCP tool surface was implemented along with:

- Product, seller, company, category, and review queries
- Validation, error handling, deduplication, and audit logging
- Shared caching and rate limiting
- Response budgeting for AI agents
- GammaMarkets-based product and review parsing
- Cursor based Pagination and sorting
- NIP-50 search
- NIP-05 support
- Nostr relay-fetch improvements
- Follow/unfollow using NIP-02
- Shared parser extraction


The main thing still open is MCP Resources and Prompts, which I plan to continue working on after the program and eventually ship as part of the MCP package on npm.

## What I Learned

The biggest takeaway from this summer was that a lot of engineering happens after the first implementation works.

Features that look straightforward on paper become much more interesting when you start testing them, looking at real relay behavior, handling races, or thinking about what an AI agent actually needs from a response.

And probably the most important part was learning how much better a design becomes when you discuss it with someone who has more context. Many of the decisions I made during the summer became clearer after talking through the trade-offs with my mentors and peers.

## What's Next

- **MCP Resources and Prompts:** The main remaining part of the project is implementing MCP Resources and Prompts. I plan to continue working on this after the program and eventually ship the completed MCP server as an npm package.

I also hope to continue contributing to Shopstr and exploring how MCP can make Nostr-based marketplaces more accessible to AI agents. The project started as an MCP server, but the work around search, pagination, relay handling, parsing, and reliability gave me a much deeper understanding of how features are actually built

If you're interested in Nostr, MCP, or building AI agents that can interact with decentralized applications, check out the Shopstr repository and the work from this project. There is still plenty to explore and improve, and contributions and feedback are always welcome.
## Acknowledgements

A big thanks to [Gautam](https://github.com/gautam-092528) and [Calvadev](https://github.com/calvadev) for making the whole experience so smooth. They were always available when I needed help, reviewed PRs quickly, and were willing to discuss even small design decisions and edge cases. More importantly, they usually gave me the space to figure things out myself rather than just telling me what to do.

Thanks to Adi and Summer of Bitcoin for providing me with this wonderful opportunity and to the Shopstr community and other contributors for all the discussions and feedback throughout the summer.

## Links

- [Shopstr Repository](https://github.com/shopstr-eng/shopstr)
- [My Work](https://github.com/shopstr-eng/shopstr/pulls?q=is%3Apr+is%3Aclosed+author%3AAryan0699)
- [Proposal](https://docs.google.com/document/d/1ZzBcuTsSqUnVgXM6WQZnArAT37KMPy0TyDpjKVF1T8w/edit?tab=t.0)