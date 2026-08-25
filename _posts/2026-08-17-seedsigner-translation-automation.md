---
layout: post
title: "Automating SeedSigner's Translation and Review Workflow"
date: 2026-08-17
author: Chaitanya Keyal
categories: [Development, Open-Source, Security, Wallets]
image: ../assets/images/blog_content/2026-08-17-seedsigner-translation-automation.png
---

## Who I am and what I worked on

I'm Chaitanya Keyal, a student at BITS Pilani, and this was my second Summer of Bitcoin. I spent it with **SeedSigner** again, an open-source, air-gapped Bitcoin signing device that runs on cheap, off-the-shelf parts. My project was to automate SeedSigner's localization workflow: the machinery that keeps the app's translatable text, its translations, and the screenshots translators review in sync across three separate repositories.

## What I set out to do

SeedSigner is translated by volunteers on Transifex, and today the plumbing around that is manual. When a developer changes a user-facing string, someone has to regenerate the translation source file (`messages.pot`) and get it to Transifex. When translators finish, someone has to pull those translations back into the repo. And when a translator wants to check their work, they have to dig through raw CI artifacts to find the screenshots their change affected.

None of that is hard, but it's the kind of routine coordination that quietly eats maintainer time and goes stale. My goal was to remove it: keep the source strings, the translations, and the review assets in sync automatically, and give translators a review experience that doesn't require GitHub expertise.

The catch is that this all has to run on pull requests, and a pull request from a fork can contain anything. So the real problem underneath the plumbing is a security one: how do you run automation triggered by untrusted contributions without ever handing that untrusted code the keys to do damage?

## What I built

The pipeline has four streams, each an independent piece:

1. **Source-string advisory.** On any PR that changes user-facing text, a comment shows exactly which translatable strings were added, removed, or changed, so reviewers and translators can see the localization impact at a glance.
2. **Post-merge sync.** After a change merges, `messages.pot` is regenerated automatically and pushed to Transifex through a bot-owned fork, so no one hand-maintains it.
3. **Translation bridge.** When Transifex commits finished translations back, they're turned into one clean pull request per language, each a single-file diff that's easy to review.
4. **Translation-PR review.** Every translation PR gets automated checks plus a per-PR web page showing the screenshots that changed, and a single status comment on the PR.

That last stream is the reviewer experience: a translator opens a PR (or one is opened for them by the bridge), and within a few minutes there's a link to a page with before/after screenshots of exactly the screens their translation touched, any overflow findings flagged, and a green or red status on whether the automated checks passed. No CI-artifact archaeology.

## The design decision that shaped everything

The hardest and most interesting part wasn't any single feature; it was the trust model. GitHub Actions has a well-known class of vulnerability where a workflow triggered by a fork PR runs untrusted code while holding write credentials, and the untrusted code simply steals them. The whole project had to be built so that couldn't happen.

The answer, applied to every stream, is a split into two workflows:

- A **read-only producer** runs in the PR's context. It's allowed to execute the untrusted contribution: it regenerates catalogs, renders screenshots, runs the checks, but it has no secrets and a read-only token. Even fully malicious PR code can't write anything or leak anything, because there's nothing there to take.
- A **trusted consumer** runs the repository's *own* code from the default branch, never the PR's. It treats everything the producer handed it as untrusted data: it validates a strict JSON-schema manifest, confirms the data actually belongs to the PR it claims to (binding the PR number to its exact commit SHA so one PR can't hijack another's comment or page), checks every file it's about to publish, escapes every piece of translated text into the page, and only then acts with write access.

A few other decisions fell out of the same principle:

- **No upstream repository is ever granted write access to its own contents.** All branch writes happen in bot-owned forks, using short-lived GitHub App tokens that are scoped down to exactly what each step needs and revoked when the job ends.
- **The Transifex integration is fenced in.** It can only write translation files to one branch of a bot fork; a path guard rejects anything else, and I confirmed the integration's own permissions can't touch workflow files.
- **Reconciliation is stateless.** Every run compares the current state against upstream and converges, so reruns and out-of-order events never produce duplicate PRs.

## What shipped

The work is 14 pull requests across the two repositories, roughly 6,000 lines, mostly tests and docs. They're collected in a [tracking issue](https://github.com/SeedSigner/seedsigner/issues/946) with a note on each so reviewers can pick ones in their comfort zone, some are plain Python with unit tests, some are GitHub Actions workflows worth a security eye, a few are docs.

[Main repo](https://github.com/SeedSigner/seedsigner):

- [#940](https://github.com/SeedSigner/seedsigner/pull/940) manifest contract + catalog diff engine
- [#941](https://github.com/SeedSigner/seedsigner/pull/941) advisory comment renderer + manifest validator
- [#942](https://github.com/SeedSigner/seedsigner/pull/942) advisory diff-comment workflows
- [#943](https://github.com/SeedSigner/seedsigner/pull/943) post-merge sync + rolling PR
- [#944](https://github.com/SeedSigner/seedsigner/pull/944) docs
- [#955](https://github.com/SeedSigner/seedsigner/pull/955) translation overflow scanner

[Translations repo](https://github.com/SeedSigner/seedsigner-translations):

- [#93](https://github.com/SeedSigner/seedsigner-translations/pull/93) locale-change detector
- [#94](https://github.com/SeedSigner/seedsigner-translations/pull/94) bridge workflow
- [#95](https://github.com/SeedSigner/seedsigner-translations/pull/95) bridge docs
- [#96](https://github.com/SeedSigner/seedsigner-translations/pull/96) placeholder integrity check
- [#97](https://github.com/SeedSigner/seedsigner-translations/pull/97) review manifest
- [#98](https://github.com/SeedSigner/seedsigner-translations/pull/98) review producer workflow
- [#99](https://github.com/SeedSigner/seedsigner-translations/pull/99) review page + status comment
- [#100](https://github.com/SeedSigner/seedsigner-translations/pull/100) review docs

I tested the whole chain end to end on my own forks with a test bot account and a free Transifex project: a real Transifex translation flowing into a per-language PR, and a translation PR getting its checks, its status comment, and a live review page. One of the checks earned its keep immediately, the placeholder check found a real bug in the current Spanish catalog, where a translation had dropped a `{}` field that the app fills in at runtime, so the screen was silently losing part of its text. I verified it with a screenshot and passed it to the translators.

## What's left, and a course correction

Nothing is merged yet, all 14 PRs are open and awaiting review. Going live also needs an official bot account and a GitHub App configured, which I've documented and offered to help set up.

There's also a deliberate change of direction on the last piece. My original plan included building a continuously-updated screenshot gallery site to review against. Partway through, SeedSigner's lead developer made strong progress on an LVGL-based screen rewrite that will eventually replace the current Python/Pillow rendering, and it already ships exactly the kind of multi-language, multi-resolution gallery and interactive renderer I'd planned to build. Rather than build a parallel version that the migration would soon obsolete, we agreed I'd build the reviewer experience on top of his work instead. That collaboration continues past the program. The `.po` translation pipeline I built is unaffected either way. The LVGL side reads from the same translation catalogs, so only the rendering-specific parts of the review will need rework when that migration lands.

## What I learned

The biggest shift was learning to treat my own automation as an attacker would. Coming in, I thought of CI as glue you write once and forget. On a Bitcoin project the standard is different: every workflow that touches a PR is a potential entry point, and the interesting engineering is in the boundaries; what runs where, what holds a secret, what's allowed to write, and how you prove a piece of data is what it claims to be before you act on it. I spent as much time on those boundaries as on the features, and by the end I ran a full adversarial audit of my own workflows and could reason concretely about why each one was safe.

The other thing I took away is how much good open-source work is about keeping changes small, independent, and reviewable. Fourteen focused PRs with clear dependencies are far more likely to get merged by a small volunteer team than one large one.

## What's next

I'll keep shepherding these PRs through review, help with the go-live setup when the maintainers are ready, and build the LVGL-based reviewer experience as that rewrite matures. I've been contributing to SeedSigner for over a year now, and I intend to keep at it well past this summer.

If you're interested in the code, the [tracking issue](https://github.com/SeedSigner/seedsigner/issues/946) is the best entry point, and reviews or ideas on any of the PRs are very welcome.
