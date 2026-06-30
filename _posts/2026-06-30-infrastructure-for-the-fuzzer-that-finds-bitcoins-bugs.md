---
layout: post
title: "Infrastructure for the Fuzzer That Finds Bitcoin's Bugs"
date: 2026-06-30
author: Suryansh Singh
categories: [Security, Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-06-30-bitcoinfuzz-orchestrator.png
---

The first time I ran `docker compose up script` and watched libFuzzer's stdout scroll past, I didn't think about the tool I was about to build. I thought about how easy it would be to lose this.

Somewhere in that stream of `cov:`, `ft:`, `exec/s:`, and `rss:` numbers was a process comparing how more than 20 different Bitcoin implementations interpret the same bytes of Bitcoin Core, rust-bitcoin, LDK, LND, BTCD, and the rest. If two of them disagreed, that disagreement would scroll past with everything else, and unless a human happened to be staring at the terminal at that exact moment, it would simply vanish into the logs. No alert. No record. No second chance to notice. The fuzzer that had already found a CVE in `rust-miniscript` and more than 80 confirmed bugs across the ecosystem was, operationally, held together by someone remembering to check on it.

That gap between what bitcoinfuzz could find and what anyone could actually see it finding is what I've spent the first six weeks of my Summer of Bitcoin mentorship trying to close.

## Why this project, specifically

I came into Summer of Bitcoin already comfortable with the pieces this project needed: Docker and container orchestration, FastAPI services with background workers, Prometheus exposition endpoints, React dashboards that don't need a page refresh to feel alive. What pulled me toward bitcoinfuzz wasn't one feature I wanted to build. It was the shape of the problem. Here was a tool doing genuinely important work, and the only thing standing between "this works" and "this is operationally trustworthy" was infrastructure. Not a new algorithm. Not a smarter fuzzer. Just the unglamorous work of orchestration, observability, and alerting.

I find that kind of problem more interesting than people give it credit for. Anyone can run one fuzz target in a terminal and watch it. The real test is running thirty of them, unattended, for days, and trusting that if one finds something, you'll know before you have to go looking.

## What I expected versus what I found

Going in, I expected the hard part to be the orchestrator itself which includes the state machine, the Docker SDK calls, the scheduler. I'd built services like that before. What I underestimated was how much of the actual difficulty would live in the seams between components, in the places where my code had to make assumptions about an environment it didn't fully control.

The existing system was deliberately minimal: a `docker-compose.yml` defining one service per fuzz target, a bind mount at `./docker:/app/data` for corpus and crash persistence, and an operator typing commands by hand. My job was never to replace that. The proposal was explicit about preserving the Docker-based build and runtime model exactly as it was but to build a layer above it that could see what the operator couldn't that is container health across all targets at once, structured metrics instead of raw log lines, and a record of what happened when nobody was watching.

That constraint, build on the system rather than around it, sounds simple in a proposal document. In practice it meant every decision had to account for an existing, working setup I wasn't allowed to disturb.

## What's shipped so far

Six weeks in, the core of that layer exists. Pull request #584 against `bitcoinfuzz:v2` lays down the orchestrator, the metrics pipeline, and the alerting backend in one submission: 8 commits, 35 files changed, roughly 5,500 lines added.

On the orchestrator side, there's now a target registry that reads `docker-compose.yml` directly, so every fuzz target the CI pipeline already knows about is automatically available with no duplicated configuration. Campaigns move through a state machine with seven states (queued, starting, running, completed, paused, crashed, restarting, and failed) persisted to a SQLite database running in WAL mode so it survives concurrent writes. If the orchestrator process dies and comes back, it re-attaches to whatever containers were still running instead of losing track of them. A resource-aware scheduler sits in front of all of this, using a priority queue so CPU and memory limits are respected even when more campaigns get requested than the machine can run at once. A crash watcher polls the same volume directories the fuzzer already writes to, and the moment a new crash artifact appears, it gets logged and routed toward an alert.

Alongside that, a metrics agent attaches to each container's log stream and parses libFuzzer's pulse output in real time which are coverage edges, feature count, corpus size, executions per second, memory usage and exposes all of it through a standard Prometheus `/metrics` endpoint. Prometheus is configured to discover scrape targets dynamically as campaigns start and stop, and I wrote Alertmanager rules for the three failure modes that matter most: a campaign crashing outright, a new crash being found, and execution rate dropping low enough to suggest something's stuck.

It's backed by 181 passing tests, including a parser test suite built against more than fifty real log lines pulled from actual campaign output deliberately including the messy edge cases, memory limit hits, crash triggers, rather than just the clean ones.

The part I'm proudest of isn't any single file. It's that the state machine and the crash watcher work together the way I'd hoped on paper. A crash isn't just detected, it's recorded with enough context that someone reviewing it later doesn't have to reconstruct what happened from raw logs.

## The hardest problem, and the trade-off underneath it

The genuinely difficult part wasn't the happy path. It was deciding what "correct" meant when the orchestrator's view of the world and Docker's actual state disagreed, specifically, what should happen to a campaign sitting in the queue when the orchestrator process restarted.

My first instinct was that this was a small bug: on startup, reload campaigns from SQLite, done. But queued campaigns turned out to be a different category of problem than running ones. A running campaign has a container you can reattach to. Docker still knows it exists. A queued campaign has no container yet; its only existence is a row in a database and, before my fix, a slot in an in-memory priority queue that restarting the process wipes clean. The campaign would reload from the database looking exactly like it was still queued, but the scheduler had no memory of it, so it would never get picked up again. Not crashed. Not failed. Just permanently, silently waiting.

That's the kind of bug that's worse than a crash, because nothing about it looks broken. The fix meant treating scheduler state as something that has to be reconstructed on startup, not just trusted to have survived which sounds obvious once you've found it and felt slightly foolish for not building it that way from the start.

Underneath that bug was a design trade-off I had to think through deliberately: how much should the orchestrator trust its own persisted state versus re-verifying everything against Docker on every restart? Re-verifying everything is safer but slower and more complex. Trusting the database is faster but exactly the kind of assumption that produces bugs like the one above. I landed on persisting state but treating Docker as the source of truth for anything currently running, and the database as the source of truth for anything that isn't. I'm still validating that compromise as the second half of the project moves into heavier load testing.

I also made a scoping call worth being honest about: the proposal specified Go for the metrics agent, and for the mid-term I built it in Python instead, still exposing a Prometheus-compatible endpoint, to keep iteration fast and the backend in one language while the orchestrator's shape was still settling. Whether that stays Python or gets ported to Go is one of the open questions heading into the second half.

## What review actually looks like here

Submitting the PR didn't feel like crossing a finish line so much as opening the actual conversation. My mentor's review caught exactly the kind of issues that don't show up until someone reads the code with production conditions in mind rather than a local demo. Four findings, specifically: environment variables defined in the Docker Compose infra file that my config loader simply wasn't reading, which meant the crash watcher would point at the wrong path the moment it ran inside a container instead of on my system; a metrics and alerting backend that was fully built but never actually wired into the FastAPI app's startup sequence, so `/metrics` wouldn't exist at runtime despite all the code for it being there; the queued-campaign bug described above; and a `max_duration_seconds` field that campaigns could set but that nothing in the health loop actually enforced, meaning a scheduled campaign with a time limit would simply run forever unless someone stopped it by hand.

None of these were design failures. They were the difference between code that works when you trace through it by hand and code that works when it's actually deployed which is precisely the gap this project exists to close, and a fairly humbling thing to be reminded of.

## What I understand now that I didn't six weeks ago

I came in thinking the hard part of monitoring infrastructure was the dashboard, the part people actually look at. I now think the dashboard is the easy part. The hard part is everywhere underneath it: making sure the thing you're measuring is still the thing you think it is after a restart, after a crash, after the orchestrator itself goes down and comes back up. Observability isn't a feature you bolt onto a working system. It's a property the system either has or doesn't, baked into how state gets stored and recovered, long before any chart gets drawn.

I also understand bitcoinfuzz itself differently now. Reading about a differential fuzzer in a proposal is one thing. Watching it actually flag a divergence between implementations, and realizing that divergence is the entire reason this project's plumbing needs to be trustworthy, is another.

## What's left for the second half

The priority I decided on is straightforward: take the four review findings from PR #584 from "identified" to "fixed and verified", then move into the parts of the proposal that mid-term deliberately deferred that are the historical query API, the React dashboard itself, and the decision on whether the metrics agent stays in Python or moves to Go. Past that, the remaining weeks are about load: running real campaigns against the orchestrator for extended periods, watching for the failure modes that only show up after hours rather than minutes, and making sure the crash-to-alert pipeline holds up under conditions less forgiving than my own test suite.

The goal by final evaluation is for an operator to launch thirty fuzz targets, walk away, and trust that if any of them finds something, they'll know, without ever needing to be the person staring at a terminal.

That's still the whole point. It was the point the first time I watched that log scroll past and realized how much was riding on someone simply paying attention. Six weeks later, the system is starting to pay attention so a person doesn't have to.

## Links to my work

- Repository: [bitcoinfuzz/bitcoinfuzz](https://github.com/bitcoinfuzz/bitcoinfuzz)
- Pull request: [Implement Campaign Orchestrator, Metrics Agent, and Alerting Backend (#584)](https://github.com/bitcoinfuzz/bitcoinfuzz/pull/584)
- My fork and branch: [devSuryansh/bitcoinfuzz:feat/orchestrator](https://github.com/devSuryansh/bitcoinfuzz/tree/feat/orchestrator)
