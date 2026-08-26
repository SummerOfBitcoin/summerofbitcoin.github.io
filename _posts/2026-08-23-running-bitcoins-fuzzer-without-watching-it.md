---
layout: post
title: "Running Bitcoin's Fuzzer Without Watching It"
date: 2026-08-23
author: Suryansh Singh
categories: [Security, Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-06-30-bitcoinfuzz-orchestrator.png
---

[bitcoinfuzz](https://github.com/bitcoinfuzz/bitcoinfuzz) is a differential fuzzer. It feeds the same input bytes to Bitcoin Core, rust-bitcoin, LDK, LND, BTCD and more than twenty other implementations, then checks whether they all agree on what those bytes mean. When two implementations disagree about a transaction, a script, or a descriptor, that disagreement is a bug in at least one of them, and possibly a consensus problem. The project has surfaced more than 80 confirmed bugs this way, including a CVE in `rust-miniscript`.

Running it looked like this: `docker compose up script`, then watch libFuzzer's stdout.

That works for one target for an afternoon. It does not work for thirty targets over a weekend. A crash at 3am is a line that scrolls past. A fuzzer that quietly wedges at 40 exec/s looks identical to one that is working, unless you happen to be reading. There was no record of which campaigns had run, no way to compare this week's coverage against last week's, and no way to find out that something happened other than being present when it did.

My Summer of Bitcoin project was the layer that closes that gap: [bitcoinfuzz-infra](https://github.com/bitcoinfuzz/bitcoinfuzz-infra), a control plane that launches campaigns, supervises them, turns the fuzzer's output into metrics, and raises an alert when something needs a human.

## The shape of the system

Three pieces, each with one job.

The **orchestrator** is a FastAPI service. It is the only thing an operator talks to. It knows which fuzz targets exist, decides when a campaign can start, launches and supervises the containers, and keeps the history.

The **metrics agent** reads a running campaign's log stream and turns libFuzzer's output into Prometheus metrics.

The **monitoring stack** is Prometheus and Alertmanager, scraping those metrics and firing alerts when the numbers say something is wrong.

The constraint I set early, and kept, was that none of this replaces how bitcoinfuzz builds or runs. The fuzzer's Dockerfile, its compose file, and its `./docker` data directory all stay exactly as they were. My layer sits above them and drives them. If the whole orchestrator disappeared tomorrow, an operator could go back to typing `docker compose up` and nothing would be broken.

## Targets: one source of truth

The first design decision was where the list of fuzz targets comes from.

The easy option is a config file in my repo listing every target. The problem is obvious the first time someone adds a target to bitcoinfuzz and forgets to add it to mine: the orchestrator silently cannot launch a target that exists.

So the target registry parses bitcoinfuzz's own `docker-compose.yml`. Every service defined there becomes a launchable target, along with the environment variables and build arguments it declares. Add a target to the fuzzer, and the orchestrator picks it up on the next restart with no second edit.

That decision got tested when the project moved into its own repository mid-program. My mentor Bruno judged that an orchestration stack, with its FastAPI and Prometheus and Docker SDK dependencies, did not belong inside the repository that Bitcoin developers actually review, and he was right. But the move meant the compose file sitting next to my code was suddenly my own infra compose file, listing the orchestrator and Prometheus rather than fuzz targets. The registry dutifully offered to launch Prometheus as a fuzzing campaign.

The fix was to make the source explicit rather than implicit: an `ORCHESTRATOR_COMPOSE_FILE` setting pointing at the fuzzer's compose file, plus filtering that excludes infra services. The broader lesson stuck with me. "Read the config from the repo" is only unambiguous while there is exactly one repo.

## The campaign lifecycle

A campaign is one fuzz target running with a particular set of modules and resource limits. It moves through eight states:

```
QUEUED -> STARTING -> RUNNING -> COMPLETED
   |                    |   ^        |
   v                    v   |        v
COMPLETED            PAUSED  |    (archive)
(user stop)             |    |
                        v    |
                     CRASHED -> RESTARTING -> RUNNING
                        |
                        v
                     FAILED (max retries exceeded)
```

Writing this as an explicit state machine rather than a status string was the single decision that paid off most. Every transition goes through one function that validates it against a table of legal moves, so a bug that would otherwise produce a campaign in a nonsensical state raises an error at the point it happens instead of surfacing three steps later as corrupt data.

It also forced me to answer questions I would have otherwise left implicit. What happens when a campaign crashes for the fourth time and `max_retries` is three? It goes to `FAILED`, not `CRASHED`, and stops consuming a restart budget. What happens when you delete a campaign that has not started yet? It leaves the queue and goes straight to `COMPLETED` with an exit reason of `stopped`, because a queued campaign has no container to kill. Those are the kinds of edge cases that turn into 3am mysteries if nobody writes down what is supposed to happen.

Creating a campaign is a single POST:

```json
{
  "target_name": "script",
  "modules": "BITCOIN_CORE,RUST_BITCOIN",
  "cpu_quota": 200000,
  "memory_mb": 2048,
  "priority": 5,
  "max_retries": 3,
  "max_duration_seconds": 86400
}
```

Around that there are 18 endpoints covering campaign control (create, list, inspect, pause, resume, delete), live log streaming over server-sent events, crash listings, cron-style scheduled campaigns, per-target history, comparison between two historical runs, and dashboard summary and resource views.

## Scheduling against a budget

You cannot run thirty fuzz targets on an eight-core box just because someone asked for thirty. Fuzzing is CPU-bound by design, and libFuzzer will happily consume whatever memory you let it.

So campaigns do not launch on request; they get admitted. The scheduler holds a budget, auto-detected as the host core count and 80% of host RAM unless configured otherwise, and tracks what is currently allocated. A campaign whose CPU and memory limits fit in the remaining budget launches immediately. One that does not goes into a priority queue and waits.

Priority is 1 to 10, and the queue is ordered by priority first, then by enqueue time, so a high-priority campaign jumps the line but two equal-priority campaigns are served fairly. Whenever a campaign finishes and its resources are released, the queue is re-examined and anything that now fits is launched.

The important property is that the budget is enforced before a container exists, not after. Letting Docker start thirty containers and then discovering the box is thrashing is not a recoverable situation on a shared host.

## Running containers from inside a container

The orchestrator ships as a container that manages other containers, which produces a category of problem I had not dealt with before and now look for everywhere.

The first is identity. The image runs as a non-root user, which is what you want for a service holding the Docker socket. But a non-root user cannot read `/var/run/docker.sock` unless it is in the group that owns it, and that group's ID is a property of the host, not the image. The compose file therefore takes a `DOCKER_GID` and adds it to the container's supplementary groups, and refuses to start if it is unset rather than failing later with a permission error.

The second is subtler and it is my favorite thing I learned this summer. My compose file mounts the fuzzer's data directory into the orchestrator at `/app/data`. The orchestrator then launches campaign containers by asking Docker to bind-mount that same data directory. The natural thing to do is pass `/app/data`, because that is what the directory is called.

That is wrong, and it fails quietly. The Docker daemon runs on the host. `/app/data` means nothing there, so the daemon does not error out; it interprets the string as a named volume and creates an empty one. The campaign runs, writes its corpus into a void, and finds nothing on the next run.

The fix is to keep the two paths separate and be explicit about which is which. The orchestrator reads corpus and crash files through its own container path, and passes a configured absolute host path (`ORCHESTRATOR_HOST_DATA_DIR`) to the daemon. Whenever a containerized process asks the host daemon to mount something, the path that process knows is not the path the daemon needs.

## From log lines to metrics

libFuzzer reports progress on stdout in a compact format:

```
#2097152 pulse  cov: 847 ft: 3201 corp: 412/2Mb exec/s: 14832 rss: 387Mb
```

Everything an operator cares about mid-campaign is in that line, and none of it is queryable. The metrics agent attaches to each campaign's log stream, parses these lines, and exposes them as Prometheus metrics on port 9091:

| Metric | What it tells you |
|---|---|
| `bitcoinfuzz_coverage_edges` | Edges hit. Flat means the fuzzer has stopped learning. |
| `bitcoinfuzz_features` | Feature count, a finer-grained view of the same thing. |
| `bitcoinfuzz_corpus_files_total` / `_bytes_total` | Corpus growth. |
| `bitcoinfuzz_exec_per_second` | Throughput. A collapse usually means contention or a wedged target. |
| `bitcoinfuzz_rss_mb` | Memory, watched against the campaign's limit. |
| `bitcoinfuzz_iteration` | Total executions. |
| `bitcoinfuzz_crash_total` | Crash artifacts found, as a monotonic counter. |
| `bitcoinfuzz_container_status` | Whether the container is actually up. |

Parsing looks trivial until you do it against real output. libFuzzer emits several line shapes beyond `pulse`, including `NEW` and `REDUCE` lines with the same statistics block, `READ` lines at startup, and event lines for crashes, out-of-memory kills, timeouts and normal completion. Sizes come as human strings like `2Mb` and `412Kb` rather than bytes. I built the parser suite against real log lines pulled from actual campaign output, deliberately including the ugly ones, because a parser tested only against clean sample lines has been tested against nothing.

Making `bitcoinfuzz_crash_total` a Prometheus `Counter` rather than a `Gauge` matters more than it sounds. Crash detection reports an absolute count of files in a directory, so the naive implementation sets a gauge to that number. But a counter is what lets Prometheus answer "did a new crash appear in the last minute", which is the question the alert actually asks, and it survives the agent restarting. The agent therefore tracks the last count it saw per campaign and increments by the delta.

## Catching crashes and raising alerts

When libFuzzer finds an input that crashes a target, it writes an artifact file into the target's crash directory. The crash watcher takes a snapshot of the files present when a campaign registers, then polls for anything new. A new file becomes a database record with its target, campaign, filename and timestamp, and bumps the crash counter that Prometheus scrapes.

There is a wrinkle worth naming, because it is a real limitation. Crash directories are keyed by target, not by campaign. If two campaigns run the same target at the same time, a single crash file cannot be attributed to both without double-counting it, so it is attributed to exactly one. Correct totals, imperfect attribution. Fixing it properly means per-campaign crash directories, which changes how the fuzzer writes files, and that was a bigger change than I wanted to make to a repository I was trying not to disturb.

Alertmanager covers five conditions:

```yaml
- alert: NewCrashFound
  expr: increase(bitcoinfuzz_crash_total{job="orchestrator"}[1m]) > 0

- alert: LowExecRate
  expr: bitcoinfuzz_exec_per_second{job="orchestrator"} < 100
  for: 5m
```

plus a campaign entering the crashed state, memory climbing toward the configured cap, and a corpus that has stopped growing. `LowExecRate` and the stalled-corpus rule are the two I would not have thought to write a year ago. They catch the failure mode where nothing has crashed and nothing has errored, but the campaign has stopped doing useful work, which is exactly the case a human staring at a terminal would notice and no naive health check ever will.

## Surviving a restart

The orchestrator is a long-running service on a shared host. It will be restarted, and campaigns started before the restart are still running afterward, because containers outlive the process that created them.

State lives in SQLite in WAL mode, across three tables: live campaigns, archived campaign summaries, and crash records. The split matters for queries. Once a campaign finishes, its aggregate result is written to a summary row, which is what per-target history and run-to-run comparison read. Those queries stay fast without scanning live campaign state.

On startup the orchestrator reloads campaigns from the database and then reconciles them against reality. My rule is that Docker is the source of truth for anything that should be running, and the database is the source of truth for everything else. A campaign the database says is `RUNNING` gets checked against the daemon: if the container is alive, the orchestrator re-attaches and resumes supervising it; if it exited while the orchestrator was down, the campaign is finalized with the exit code the container actually reported.

Queued campaigns are the case I nearly got wrong. A running campaign has a container to find. A queued campaign has nothing but a database row, because its place in the scheduler queue lived in memory and the restart erased it. Reload it naively and it looks perfectly healthy, still marked queued, while the scheduler has no idea it exists. It waits forever. Nothing about it looks broken, which is what makes it the worst kind of bug. Scheduler state has to be rebuilt from the database on startup, not assumed to have survived.

## Running it

The whole stack comes up with one command:

```bash
cp .env.example .env   # set BITCOINFUZZ_REPO, DOCKER_GID, host data dir
docker compose up --build -d
```

That gives you the API on 8000, metrics on 9091, Prometheus on 9090 and Alertmanager on 9093. The documentation covers both deployments I built for: a shared team host where several operators drive one orchestrator, and a single workstation with the two repositories checked out side by side.

## Where the project stands

The control plane is built and works: campaign lifecycle, scheduling, Docker management, crash detection, metrics, alerting, persistence, history and comparison queries, cron-scheduled campaigns, CI, and setup documentation. It is all in [pull request #1](https://github.com/bitcoinfuzz/bitcoinfuzz-infra/pull/1) against bitcoinfuzz-infra, which is open with review feedback I am currently working through, most importantly tightening the network defaults so the API is not reachable off-host without an explicit opt-in.

The React dashboard in my project title is the piece I did not deliver. What exists is the API it was designed to consume, including summary, resource, history, comparison and live-log endpoints. The second half of the program went into moving the codebase into its own repository and then into review cycles on the backend, and I kept servicing one large pull request instead of opening the frontend workstream beside it. They were independent and I could have done both. If I were planning this again I would split the backend into three smaller pull requests early, so that review on one piece did not block progress on everything else.

## What I take away

I came into this thinking the hard part of monitoring infrastructure was the dashboard, the part people look at. I now think it is the least interesting part.

The hard part is state. Making sure the thing you are measuring is still the thing you think it is after a restart, after a crash, after the supervisor itself goes down and comes back. Observability is not a feature you attach to a working system; it is a property the system either has or does not, decided by how state is stored and recovered long before any chart is drawn.

The other thing I learned is where the limits of my own testing are. My test suite runs without Docker and without a bitcoinfuzz checkout, which keeps it fast and worth running. But every operational bug in this project, the socket group, the bind-mount path, the ones still open, was found by somebody installing the stack on a machine that was not mine. A mock will confirm you called the right function with the right arguments. It will never tell you the host disagrees with you.

Next is finishing the review on PR #1, landing it, and then building the dashboard on top of an API that is already sitting there waiting for it.

Thanks to Bruno for reviewing by actually running the thing, for the repository call I should have made myself, and for bringing in a second reviewer rather than being the only pair of eyes on it. Thanks to Erick Cestari for the most useful code review I have received.

## Links

- Repository: [bitcoinfuzz/bitcoinfuzz-infra](https://github.com/bitcoinfuzz/bitcoinfuzz-infra)
- Final pull request (open, in review): [Initialize Orchestrator Service and Prometheus/Alertmanager Monitoring Stack (#1)](https://github.com/bitcoinfuzz/bitcoinfuzz-infra/pull/1)
- Superseded proof-of-concept PR: [Implement Campaign Orchestrator, Metrics Agent, and Alerting Backend (#584)](https://github.com/bitcoinfuzz/bitcoinfuzz/pull/584)
- Mid-term post: [Infrastructure for the Fuzzer That Finds Bitcoin's Bugs](https://blog.summerofbitcoin.org/infrastructure-for-the-fuzzer-that-finds-bitcoins-bugs/)
- The fuzzer itself: [bitcoinfuzz/bitcoinfuzz](https://github.com/bitcoinfuzz/bitcoinfuzz)
