---
layout: post
title: "Infrastructure for the Fuzzer That Finds Bitcoin's Bugs"
date: 2026-06-30
author: Suryansh Singh
categories: [Security, Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-06-30-bitcoinfuzz-orchestrator.png
---

The first time I ran `docker compose up script` and watched libFuzzer's stdout scroll past, I didn't think about the tool I was about to build. I thought about how easy it would be to lose this.

Somewhere in that stream of `cov:`, `ft:`, `exec/s:`, and `rss:` numbers was a process comparing how Bitcoin Core, rust-bitcoin, LDK, LND, BTCD, and more than twenty other Bitcoin implementations interpret the same input bytes. If two of them disagreed, that disagreement would scroll past with everything else, and unless a human happened to be staring at the terminal at that exact moment, it would simply vanish into the logs. No alert. No record. No second chance to notice. The fuzzer that had already found a CVE in `rust-miniscript` and more than 80 confirmed bugs across the ecosystem was, operationally, held together by someone remembering to check on it.

That gap between what bitcoinfuzz could find and what anyone could actually see it finding is what I've spent the first six weeks of my Summer of Bitcoin mentorship trying to close.

## Why this project, specifically

I came into Summer of Bitcoin already comfortable with the pieces this project needed: Docker and container orchestration, FastAPI services with background workers, Prometheus exposition endpoints, and React dashboards that don't need a page refresh to feel alive. What pulled me toward bitcoinfuzz wasn't one feature I wanted to build; it was the shape of the problem. Here was a tool doing genuinely important work, and the only thing standing between "this works" and "this is operationally trustworthy" was infrastructure. Not a new algorithm. Not a smarter fuzzer. Just the unglamorous work of orchestration, observability, and alerting.

I find that kind of problem more interesting than people give it credit for. Anyone can run one fuzz target in a terminal and watch it. The real test is running thirty of them, unattended, for days, and trusting that if one finds something, you'll know before you have to go looking.

## What I expected versus what I found

Going in, I expected the hard part to be the orchestrator itself: the state machine, the Docker SDK calls, the scheduler. I'd built services like that before. What I underestimated was how much of the actual difficulty would live in the seams between components, in the places where my code had to make assumptions about an environment it didn't fully control.

The existing system was deliberately minimal: a `docker-compose.yml` defining one service per fuzz target, a bind mount at `./docker:/app/data` for corpus and crash persistence, and an operator typing commands by hand. My job was never to replace that. The proposal was explicit about preserving the Docker-based build and runtime model exactly as it was, while building a layer above it that could see what the operator couldn't: container health across all targets at once, structured metrics instead of raw log lines, and a record of what happened when nobody was watching.

The constraint to build on the system rather than around it sounds simple in a proposal document. In practice, it meant every decision had to account for an existing, working setup I wasn't allowed to disturb.

## What I completed in the first six weeks

Six weeks in, the core of that layer exists. The work now lives in a dedicated repository, [bitcoinfuzz/bitcoinfuzz-infra](https://github.com/bitcoinfuzz/bitcoinfuzz-infra), with [pull request #1](https://github.com/bitcoinfuzz/bitcoinfuzz-infra/pull/1) currently open and under review. The earlier proof-of-concept PR ([bitcoinfuzz/bitcoinfuzz#584](https://github.com/bitcoinfuzz/bitcoinfuzz/pull/584), now closed) established the initial design; the infra repository is where the implementation lives going forward. The PR introduces the orchestrator, the metrics pipeline, and the alerting backend together.

On the orchestrator side, there is now a target registry that reads `docker-compose.yml` directly, so every fuzz target the CI pipeline already knows about is automatically available with no duplicated configuration. Campaigns move through a state machine with eight states (queued, starting, running, completed, paused, crashed, restarting, and failed), persisted to a SQLite database running in WAL mode so it survives concurrent writes. If the orchestrator process dies and comes back, it re-attaches to whatever containers were still running instead of losing track of them. A resource-aware scheduler sits in front of all of this, using a priority queue so CPU and memory limits are respected even when more campaigns are requested than the machine can run at once. A crash watcher polls the same volume directories the fuzzer already writes to, and the moment a new crash artifact appears, it gets logged and routed toward an alert.

Alongside that, a metrics agent attaches to each container's log stream and parses libFuzzer's pulse output in real time. It extracts coverage edges, feature count, corpus size, executions per second, and memory usage, then exposes all of it through a standard Prometheus `/metrics` endpoint. Prometheus is configured to discover scrape targets dynamically as campaigns start and stop. I also wrote Alertmanager rules for the three failure modes that matter most: a campaign crashing outright, a new crash being found, and execution rate dropping low enough to suggest something is stuck.

The test suite has 181 passing tests, including a parser suite built against more than fifty real log lines pulled from actual campaign output. Those lines were chosen deliberately to include messy edge cases, including memory limit hits and crash triggers, rather than just the clean, well-formed ones.

The part I'm proudest of isn't any single file. It's that the state machine and the crash watcher work together the way I'd hoped on paper. A crash isn't just detected, it's recorded with enough context that someone reviewing it later doesn't have to reconstruct what happened from raw logs.

## The hardest problem, and the trade-off underneath it

The genuinely difficult part wasn't the happy path. It was deciding what "correct" meant when the orchestrator's view of the world and Docker's actual state disagreed, specifically what should happen to a campaign sitting in the queue when the orchestrator process restarted.

My first instinct was that this was a small bug: on startup, reload campaigns from SQLite, done. But queued campaigns turned out to be a different category of problem than running ones. A running campaign has a container you can re-attach to; Docker still knows it exists. A queued campaign has no container yet. Its only existence is a row in a database and, before my fix, a slot in an in-memory priority queue that restarting the process wipes clean. The campaign would reload from the database looking exactly like it was still queued, but the scheduler had no memory of it, so it would never get picked up again. Not crashed. Not failed. Just permanently, silently waiting.

That's the kind of bug that's worse than a crash, because nothing about it looks broken. The fix meant treating scheduler state as something that has to be reconstructed on startup, not just trusted to have survived. That sounds obvious once you've found it, and it felt slightly foolish not to have built it that way from the start.

Underneath that bug was a design trade-off I had to think through deliberately: how much should the orchestrator trust its own persisted state versus re-verifying everything against Docker on every restart? Re-verifying everything is safer but slower and more complex. Trusting the database is faster but exactly the kind of assumption that produces bugs like the one above. I landed on persisting state but treating Docker as the source of truth for anything currently running, and the database as the source of truth for anything that isn't. I'm still validating that compromise as the second half of the project moves into heavier load testing.

I also made a scoping call worth being honest about: the proposal specified Go for the metrics agent, and for the mid-term I built it in Python instead. It still exposes a Prometheus-compatible endpoint, and keeping the backend in one language helped while the orchestrator's shape was still settling. Whether it stays in Python or gets ported to Go is one of the open questions heading into the second half.

## What review actually looks like here

Submitting the PR didn't feel like crossing a finish line so much as opening the actual conversation. My mentor's review of the earlier draft caught exactly the kinds of issues that don't show up until someone reads the code with production conditions in mind, not a local demo. There were four specific findings.

First, environment variables defined in the Docker Compose infra file weren't being read by the config loader. That meant the crash watcher would point at the wrong path the moment it ran inside a container instead of on my development machine. Second, the metrics and alerting backend was fully built but never wired into the FastAPI app's startup sequence, so `/metrics` wouldn't exist at runtime despite all the code for it being present. Third, the queued-campaign bug I described above. Fourth, a `max_duration_seconds` field that campaigns could set but that nothing in the health loop actually enforced: a scheduled campaign with a time limit would simply run forever unless someone stopped it by hand.

None of these were design failures. They were the difference between code that works when you trace through it by hand and code that works when it's actually deployed. That gap is precisely what this project exists to close, and being reminded of it was humbling in the most useful way.

## What I understand now that I didn't six weeks ago

I came in thinking the hard part of monitoring infrastructure was the dashboard, the part people actually look at. I now think the dashboard is the easy part. The hard part is everywhere underneath it: making sure the thing you're measuring is still the thing you think it is after a restart, after a crash, after the orchestrator itself goes down and comes back up. Observability isn't a feature you bolt onto a working system. It's a property the system either has or doesn't, baked into how state gets stored and recovered, long before any chart gets drawn.

I also understand bitcoinfuzz itself differently now. Reading about a differential fuzzer in a proposal is one thing. Watching it actually flag a divergence between implementations, and realizing that divergence is the entire reason this project's plumbing needs to be trustworthy, is another.

## What's left for the second half

The immediate priority is getting [bitcoinfuzz-infra#1](https://github.com/bitcoinfuzz/bitcoinfuzz-infra/pull/1) merged. The review is in progress, and I'm working through the feedback as it comes in. Once the PR lands, the next phase covers the parts the mid-term deliberately deferred: the historical query API, the React dashboard, and the decision on whether the metrics agent stays in Python or moves to Go.

Past that, the remaining weeks are about load: running real campaigns against the orchestrator for extended periods, watching for failure modes that only appear after hours rather than minutes, and making sure the crash-to-alert pipeline holds up under conditions less forgiving than my own test suite.

The goal by final evaluation is for an operator to launch thirty fuzz targets, walk away, and trust that if any of them finds something, they'll know, without ever needing to be the person staring at a terminal.

That's still the whole point. It was the point the first time I watched that log scroll past and realized how much was riding on someone simply paying attention. Six weeks later, the system is starting to pay attention so a person doesn't have to.

## Links to my work

- Repository: [bitcoinfuzz/bitcoinfuzz-infra](https://github.com/bitcoinfuzz/bitcoinfuzz-infra)
- Current pull request (open, under review): [Initialize Orchestrator Service and Prometheus/Alertmanager Monitoring Stack (#1)](https://github.com/bitcoinfuzz/bitcoinfuzz-infra/pull/1)
- Closed proof-of-concept PR: [Implement Campaign Orchestrator, Metrics Agent, and Alerting Backend (#584)](https://github.com/bitcoinfuzz/bitcoinfuzz/pull/584)
- My fork: [devSuryansh/bitcoinfuzz-infra](https://github.com/devSuryansh/bitcoinfuzz-infra/tree/feat/orchestrator-monitoring-setup)
