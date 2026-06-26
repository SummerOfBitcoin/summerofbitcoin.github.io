---
layout: post
title: "From Rejection to Jam: My Summer of Bitcoin Journey So Far"
date: 2026-06-27
author: Parth Bandwal
categories: [Jam, Stories, Open-Source, Development]
image: ../assets/images/blog_content/2026-06-27-parth-bandwal-sob-jam-journey.png
---

Last year, I applied to Summer of Bitcoin with Jam in mind and did not get
selected.

That sentence looks simple now, but at the time it felt heavy. I had spent
time reading about the program, looking at Jam, trying to understand how a
Bitcoin privacy wallet fit together, and imagining what it would be like to
work on open source with real maintainers. When the result came, it was not
the one I wanted.

But rejection has a strange way of becoming useful if you do not run away
from it. I could have treated it as proof that I was not ready for Bitcoin
open source. Instead, after the initial disappointment, I started treating it
as feedback: I liked the right organization, but I had not yet shown enough
evidence that I could contribute meaningfully to it.

This year, I came back to the same organization with a different mindset.

## Why Jam Stayed In My Head

Jam stood out to me because it was not just another frontend project. It is a
web interface around JoinMarket, and that means the UI is connected to real
Bitcoin privacy workflows: wallets, UTXOs, collaborative transactions,
sweeps, fidelity bonds, earning, logs, settings, and many small details that
matter when users are handling money.

The more I explored it, the more I realized that Jam needed the kind of work
I enjoy: making flows clearer, making interfaces more reliable, improving
tests, and turning rough edges into something a user can trust.

That is what pulled me back after last year's rejection. I did not want to
randomly pick a new project just because I had failed once. I wanted to come
back to Jam better prepared.

## What Changed This Time

The biggest change was that I stopped thinking of the application as the
starting point. Open source does not begin when you submit a proposal. It
begins when you clone the repo, read the code, break the app locally, ask a
question, open a small PR, receive review, and come back with a better patch.

This time, before and during the proposal phase, I focused on building that
proof slowly:

- I spent more time reading existing issues and pull requests.
- I tried to understand why maintainers preferred certain patterns.
- I worked on smaller fixes instead of waiting for one perfect big idea.
- I paid attention to testing, accessibility, translations, and edge cases.
- I learned to explain my changes clearly in PR descriptions.

That last point mattered more than I expected. A merged PR is not just code.
It is also communication. It says: here is the problem, here is what changed,
here is how I tested it, and here is the tradeoff I am aware of.

## Getting Selected

When I got selected for Summer of Bitcoin 2026 with Jam, it felt different
from a normal achievement. It felt like the previous year had finally folded
back into the story.

The same organization that had been part of my rejection became the
organization where I would contribute as an SoB intern. That made the
selection feel less like a lucky break and more like a reminder that progress
often happens quietly before anyone else sees it.

I was excited, obviously. But I also felt a lot of responsibility. Jam is not
a toy app. It sits close to Bitcoin privacy and user funds, so even UI work
has to be careful. A confusing message, stale state, bad validation, or
missing test can turn into a real user problem.

## Contributions So Far

At the time of writing, I have 49 merged pull requests in
[joinmarket-webui/jam](https://github.com/joinmarket-webui/jam). Some were
small cleanup PRs. Some touched important user flows. Some were about tests
and project infrastructure. Together, they taught me how a real open-source
codebase grows through steady, reviewed work.

One of my early merged fixes was
[PR #1036](https://github.com/joinmarket-webui/jam/pull/1036), where I fixed
an auth refresh loop that continued after logout. It looked small, but it was
the kind of bug that makes you think carefully about app state: login should
refresh tokens, logout should stop that background work cleanly.

Another small but important fix was
[PR #1039](https://github.com/joinmarket-webui/jam/pull/1039), where
`btcToSats` was changed to truncate values beyond 8 decimals instead of
rounding into fractional sats. That PR made me appreciate how Bitcoin apps
leave very little room for casual number handling. A tiny utility function
can still carry real correctness expectations.

From there, I moved into larger v2 work:

- [PR #1069](https://github.com/joinmarket-webui/jam/pull/1069) worked on the
  v2 import wallet flow.
- [PR #1081](https://github.com/joinmarket-webui/jam/pull/1081) improved the
  onboarding experience on login.
- [PR #1093](https://github.com/joinmarket-webui/jam/pull/1093) made the v2
  Sweep flow functional with modular UX and validations.
- [PR #1136](https://github.com/joinmarket-webui/jam/pull/1136) implemented
  the collaborative CoinJoin send flow in the send page.
- [PR #1159](https://github.com/joinmarket-webui/jam/pull/1159) added an
  earnings report sheet and parser.
- [PR #1164](https://github.com/joinmarket-webui/jam/pull/1164) added an
  orderbook offer summary.
- [PR #1181](https://github.com/joinmarket-webui/jam/pull/1181) added UTXO
  selection in send with address-grouped freeze and unfreeze behavior.

I also spent a lot of time on the parts of a project that users may never
directly notice but maintainers definitely feel. In
[PR #1247](https://github.com/joinmarket-webui/jam/pull/1247), I added core
component stories for Storybook. In
[PR #1252](https://github.com/joinmarket-webui/jam/pull/1252), I helped set
up Storybook preview deployment to GitHub Pages so the UI could be reviewed
more easily.

The most satisfying testing milestone so far was
[PR #1276](https://github.com/joinmarket-webui/jam/pull/1276). It expanded
test coverage across components, hooks, and context providers, covering
loading states, error states, empty states, conditional branches, and user
interactions. The combined unit and Storybook coverage reached 90.6% for
statements, 90.7% for lines, 89.1% for functions, and 82.7% for branches.

That PR felt like a mid-term checkpoint. Not because coverage numbers are the
goal by themselves, but because good tests let future contributors move
faster with more confidence.

## A Note About My Mentor, TBK

One of the best parts of this journey has been working with my mentor,
[TBK](https://github.com/theborakompanioni).

He has always been available to work through things with me, not just by
dropping suggestions and disappearing, but by actually helping me think. When
I was stuck on project decisions, Bitcoin concepts, UI tradeoffs, or review
feedback, he gave me space to reason through the problem instead of simply
handing me an answer.

What I appreciate even more is that his guidance was never limited to the
project. Many conversations went beyond Jam and Bitcoin into how to approach
real-life decisions, how to stay calm when things feel confusing, and how to
grow as a person while growing as a developer. That kind of mentorship is
rare. It made me feel supported not only as an intern, but as someone trying
to become more thoughtful, disciplined, and confident.

## What The Review Conversations Taught Me

The Jam review conversations changed how I think about code.

Before contributing seriously, I used to think a PR was successful if the
feature worked on my machine. Now I understand that maintainers are looking
at a wider picture:

- Does this match the direction of the project?
- Will this still make sense when someone else touches it later?
- Are the error states clear?
- Is the copy translatable?
- Does the test describe behavior or just implementation?
- Is this a user problem, a developer problem, or both?

That kind of review can feel slow at first, but it is where the real learning
happens. The comments pushed me to split work better, write clearer PR
summaries, test edge cases, and avoid treating UI polish as something
separate from reliability.

Jam also taught me that open source is not about proving you already know
everything. It is about being reliable while you learn. Show up, ask better
questions, accept review, improve the patch, and keep going.

## What Rejection Actually Gave Me

Looking back, not getting selected last year was painful, but it gave me a
clean signal: interest was not enough.

I needed to become the kind of contributor whose proposal was backed by
evidence. This year, I had code, review discussions, merged PRs, and a much
better understanding of Jam's product direction. That made my application
stronger, but more importantly, it made me more useful to the project.

If someone is applying to Summer of Bitcoin in the future, this is the advice
I would give:

- Pick an organization you would still care about even if selection was not
  guaranteed.
- Start contributing before the proposal round.
- Do not underestimate small PRs.
- Read review comments carefully, including comments on other people's PRs.
- Write proposals that are grounded in the codebase, not just in excitement.
- Treat rejection as data, not as a final verdict.

## What Comes Next

There is still a lot I want to improve in Jam. Right now, I am especially
interested in mobile responsiveness, shared UI consistency, better user
feedback, and making the v2 experience feel more complete across the app.

The journey so far has already changed me. Last year, Jam was the
organization I could not get into. This year, it is the project where I am
learning how real Bitcoin open source work feels: careful, collaborative,
sometimes confusing, often humbling, and deeply rewarding.

I am grateful to Summer of Bitcoin, the Jam maintainers, and everyone who
reviewed, questioned, corrected, and encouraged my work. The rejection was
part of the path. Getting selected was another part. The real journey is
showing up every week and becoming a better contributor than I was before.
