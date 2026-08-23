---
layout: post
title: "SoB Final Blog by Aditya Sarna for the Bitcoin Design Community"
date: 2026-08-23
author: Aditya Sarna
categories: [Design, Stories, AI, UXR]
---

# SoB Final Blog
By Aditya Sarna for the Bitcoin Design Community

The [Bitcoin Design Guide](https://bitcoin.design/) has been a shared resource for wallet and Lightning builders for years. In 2026, the question sitting in front of the Bitcoin Design Community is no longer "does a guide exist?" It is whether that guide still matches how people actually build.

Most of the builders we care about now work with AI in the loop. A lot of teams ship without a designer in the room. Narrative documentation that reads well for a human does not always survive contact with a context window. [Issue #1221](https://github.com/BitcoinDesign/Guide/issues/1221) frames this as a UXR-driven rethink: talk to builders first, then let research decide what the next version of the guide should be.

I joined this project through Summer of Bitcoin, working with Mogashni (Mo) and the Bitcoin Design Community. This post is a record of what I set out to do, what I actually built, the decisions that changed the work, what is live, and what is ahead.

## What I set out to do

The brief sounded simple. Help the community understand how Bitcoin and Lightning wallet builders work today, especially those using AI, and turn that understanding into a strategy for the guide.

In practice, that meant answering a more specific set of questions:

- How do builders actually start?
- Where do ideas come from, before and during building?
- Which tools and resources do they use, and why those ones?
- What does the workflow look like when it is going well, and what happens when it breaks?
- How are design decisions made when there is often no designer on the team?

I came in with a fairly product-shaped instinct. If the guide is going to become more useful in an AI-first workflow, we might eventually need more than rewritten articles. We might need executable resources, structured constraints, playgrounds, or MCP-compatible baselines that an agent can actually use. I still think that direction is worth testing. The mistake would have been to treat it as the answer before we had spoken to enough builders.

So the first job was not to redesign the guide. It was to build the research layer that would decide what "redesign" even means, and to leave behind reference points the community can keep building on.

## What I built

The work split into three layers: prototypes of what a future guide *could* look like, the research process that would decide what it *should* look like, and the outreach needed to talk to the people who would use it.

### An MCP server and a UI playground

The MCP server is the stronger technical reference point for where the guide might go. It is not the new guide. It is a foundation we can keep adding to as we learn what has to change in the existing one, what current research already shows, and what developers tell us in interviews. A user can download a JSON file, configure a local server once, and then use the screens as baselines for prompting, UI analysis, or implementation.

The UI playground sits next to it as a practical reference for an interactive version of the same idea. You pick a wallet screen from a template, change the UI visually, watch the code update, and copy the result. Together, they make the "docs plus playground plus MCP" argument concrete, instead of leaving it as a slide.

I recorded walkthroughs of both:

- [MCP setup](https://youtu.be/SPjEZ5sfvcM)
- [UI playground](https://youtu.be/FMbTaqKw-2A)

### Research inputs, then a draft of the findings

Spiral's AI survey gave the project a directional baseline. I also asked whether we should look at public conversations, especially Reddit, before interviewing. Mo's response was to get the raw material in one place and bucket it later. Combined with interviews still to come, that gave us three research points instead of one.

The scrape was noisy, as expected. Volume was high, source diversity was uneven, and builder profiles were often impossible to assign cleanly. The percentages that fall out of a scrape are not ecosystem-wide statistics. They are signals from the sample we could see.

What was more useful than the numbers was the pattern language that kept repeating.

**How design decisions get made.** Shipped products are the default design system. When a team gets stuck, the question is often "what did Phoenix, Muun, or Breez do?" rather than "what did our research say?" Architecture is usually decided first — custody model, SDK, protocol constraints — and UX is inherited from those choices. Most teams do not have a dedicated designer. Production wallets, community comments, and ad-hoc AI review fill that gap.

**How building actually starts.** Casual and indie builders often begin from a personal pain point, not a market study. Students build for a portfolio or a course. Protocol engineers often start from a spec gap. A lot of people build privately, then validate in public. Conviction about Bitcoin's properties does a lot of the work that user interviews would do in another industry. During building, the original idea rarely survives Lightning's infrastructure constraints unchanged.

**What resources fail to do.** Builders know the reference material exists. The format often does not fit the workflow. A narrative document is hard to paste into a context window and treat as a set of constraints. Experienced developers keep a private layer — system prompts, anti-patterns, version-specific gotchas — on top of public docs. Casual developers lean on the model's prior knowledge, which still defaults to web2 assumptions.

**Where people get stuck.** Context has to be re-established at the start of almost every AI session. The common model failures are not random. They are predictable category errors. Verification is a real bottleneck for people who are vibe-coding their way through a first wallet.

Those observations now live in a working [research findings draft](https://docs.google.com/document/d/1vPzbAUu3RBSo2NI8u1gg1AlBPBDvuVWE9JJQ7hZg3HI/edit?usp=sharing) that consolidates Spiral, Reddit, and research already done by the Bitcoin Design Community. They also shaped the [interview questions](https://drive.google.com/file/d/1YKDsnMhrKgahRqbY6NIgBj7l6ZFzeMP9/view?usp=sharing), so the questions would function as probes rather than as answers we had already decided were true.

We then chose [Jobs to Be Done](https://bitcoinresearch.xyz/jobs-to-be-done), from the Bitcoin UX Research Toolkit, as the framework for structuring that research. The point is not "would you use an MCP server?" The point is: what job does the next resource have to do for a builder in the middle of a real workflow, and what format can actually perform that job?

### Outreach copy, and messages that are actually specific

Later in the summer the work became more outward-facing. I spent time on positioning before Mo worked on the AI-generated copy. The angle that held was the community one: built by builders, for builders, with the guide treated as something the community can help shape. The landing page and outreach script are live at [bdg-uxr-landing.vercel.app](https://bdg-uxr-landing.vercel.app/).

Broadcast CTAs were deprioritized. One-to-one messages to wallet teams, grounded in something specific they had recently shipped, became the plan. I researched recent work across open-source Bitcoin and Lightning wallets and wrote [customised outreach](https://docs.google.com/document/d/1xHQZdG-NdWd6eBi3ihw0tOeQOmtJ140l1bbQ9nK1GAQ/edit?tab=t.0) for each team so the first message would not sound like a generic research invite.

The Figma file started as social images and hook lines. It is now a broader [social media research base](https://www.figma.com/design/2QxoHjwCShS9ssiEQ7DLPs/Untitled?node-id=0-1&t=gipNRZ3uIJE73B8X-0): a larger set of posts the team can use both to talk about the work in public and as input for further research.

## Major technical and design decisions

A few decisions mattered more than the rest of the output.

**Build reference points, then fill them with research.** The MCP server and the playground shipped as foundations, not as the strategy. They give the community something real to extend once interviews and the existing-guide reviews tell us what has to change.

**Ten human questions over forty comprehensive ones.** A large question bank is useful as raw material. It is a poor interview. The questions that survived were the ones you would ask if you sat down next to someone about to start a wallet and tried to understand their week, not their stack.

**Jobs to be done, not feature preference.** Once the interview script tightened, we used JTBD to keep the research aimed at the job the next resource has to perform, and the format that can perform it.

**Warm outreach over a loud launch.** Social posts still matter, but they are there to show that the community is doing this work. The interviews themselves are more likely to come from paying attention to a specific team and asking them directly.

**Start from what already exists.** Skill-file structure, design-review skill, and reviews of the current guide UI kept the prototypes attached to the resource we are actually trying to change.

## What shipped

1. **MCP server.** A technical reference point for transforming the Design Guide. It is a foundation we can keep building on as we fold in changes to the existing guide, findings from current research, and what developers tell us. [Walkthrough](https://youtu.be/SPjEZ5sfvcM).
2. **UI playground.** A practical reference for how a transformed guide could work as an interactive experience. [Walkthrough](https://youtu.be/FMbTaqKw-2A).
3. **Developer outreach copywriting script.** A landing page and outreach resource for talking to developers and gathering insights: [bdg-uxr-landing.vercel.app](https://bdg-uxr-landing.vercel.app/).
4. **Interview questions draft.** A structured set of questions for developer interviews and user research: [questions](https://drive.google.com/file/d/1YKDsnMhrKgahRqbY6NIgBj7l6ZFzeMP9/view?usp=sharing).
5. **Expanded social media research base.** A broader collection of social posts that can feed further research and analysis: [Figma file](https://www.figma.com/design/2QxoHjwCShS9ssiEQ7DLPs/Untitled?node-id=0-1&t=gipNRZ3uIJE73B8X-0).
6. **Jobs to Be Done research framework.** The methodology we chose for structuring and reading the research: [bitcoinresearch.xyz/jobs-to-be-done](https://bitcoinresearch.xyz/jobs-to-be-done).
7. **Open-source wallet outreach messages.** Customised messages for specific wallet projects, so outreach is grounded in what those teams have actually been shipping: [outreach doc](https://docs.google.com/document/d/1xHQZdG-NdWd6eBi3ihw0tOeQOmtJ140l1bbQ9nK1GAQ/edit?tab=t.0).
8. **Research findings draft.** A working draft that consolidates findings from Spiral, Reddit, and research already done by the Bitcoin Design Community: [findings](https://docs.google.com/document/d/1vPzbAUu3RBSo2NI8u1gg1AlBPBDvuVWE9JJQ7hZg3HI/edit?usp=sharing).
9. **MCP and skill-file exploration.** An initial working understanding of MCP and skill-file structure, taken into a community call and checked there.
10. **Existing UI design reviews.** Review documents on the current guide UI and resources, including Christoph Ono's [bitcoin wallet design-review skill](https://github.com/GBKS/bitcoin-ui-gallery/blob/main/skills/bitcoin-wallet-design-review/SKILL.md).

## What I learned

Most of what I learned this summer came from working with Mo. The first real course correction was to stop treating research as a pile of questions and prototypes, and start writing from the point of view of a builder about to start a wallet. Sit in their shoes. Ask fewer, more human questions. Do not say more than the evidence can hold.

I also learned how this kind of work actually moves. You take a step in the open, someone sharpens it, and the project goes forward. That's how open source leverages community!

## What happens next

The next step is to keep talking to builders. Those conversations should sharpen what we already have, and sit alongside the other research we have already done.

After that comes the work the summer was aimed at: take those insights and implement them, so the guide can actually adapt to an AI-first way of building.

## Links

- [Bitcoin Design Guide UXR issue](https://github.com/BitcoinDesign/Guide/issues/1221)
- [MCP walkthrough](https://youtu.be/SPjEZ5sfvcM)
- [UI playground walkthrough](https://youtu.be/FMbTaqKw-2A)
- [Landing page and outreach](https://bdg-uxr-landing.vercel.app/)
- [Interview questions draft](https://drive.google.com/file/d/1YKDsnMhrKgahRqbY6NIgBj7l6ZFzeMP9/view?usp=sharing)
- [Social media research base](https://www.figma.com/design/2QxoHjwCShS9ssiEQ7DLPs/Untitled?node-id=0-1&t=gipNRZ3uIJE73B8X-0)
- [Jobs to Be Done](https://bitcoinresearch.xyz/jobs-to-be-done)
- [Wallet outreach messages](https://docs.google.com/document/d/1xHQZdG-NdWd6eBi3ihw0tOeQOmtJ140l1bbQ9nK1GAQ/edit?tab=t.0)
- [Research findings draft](https://docs.google.com/document/d/1vPzbAUu3RBSo2NI8u1gg1AlBPBDvuVWE9JJQ7hZg3HI/edit?usp=sharing)
- [Bitcoin wallet design-review skill](https://github.com/GBKS/bitcoin-ui-gallery/blob/main/skills/bitcoin-wallet-design-review/SKILL.md)
