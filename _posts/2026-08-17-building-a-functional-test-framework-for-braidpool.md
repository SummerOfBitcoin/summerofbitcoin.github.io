---
layout: post
title: "Building a Functional Test Framework for Braidpool"
date: 2026-08-17
author: Kushagra Kinra
categories: [Stories]
---

I want to start with the moment things clicked. I had been reading about Bitcoin mining for a while, not just how it works, but who controls it. The economics of it. You have these massive pools that every miner connects to, trusting a single operator to count their shares honestly, decide payouts, and not skim anything off the top. The decentralization of the network itself is real, but the mining layer on top of it is surprisingly centralized. A few pools could, if they coordinated, have a serious conversation about the chain.

When I came across Braidpool, it was one of those moments where you read a project description and think: this is the right way to think about the problem. The core idea is simple to state, even if hard to build. Instead of miners reporting to a central pool operator, they submit shares directly to each other over a peer-to-peer network. Payouts are computed from on-chain data and a DAG structure called a braid, where each valid share is a bead. Nobody trusts nobody. The math is public. I wanted to work on this.

I applied for Summer of Bitcoin 2026 with Braidpool as my first choice and got in. My project was to build the functional test framework for the node software. The goal, on paper, sounded modest: write some testing infrastructure. What I did not fully appreciate until I was a few weeks in was how much engineering goes into making a test framework that does not lie to you.

---

## Why testing Braidpool is hard

The Braidpool node is a Rust binary. It does not run in isolation. When it starts up, it immediately needs to connect to a Bitcoin Core node via a low-level socket interface to get block templates, submit mined blocks, and watch for new chain tips. It speaks to miners over Stratum. It maintains peer connections with other Braidpool nodes over its own protocol. If you want to test whether two nodes actually propagate a bead to each other, you need all of this running, for real, at the same time.

That rules out mocking. There is no meaningful sense in which you can fake a Bitcoin node at the socket level and test Braidpool behavior. The test needs a real Bitcoin Core process, a real Braidpool process, and a way to control and observe both. This kind of test, where actual compiled binaries are started and coordinated, is called a functional test, and writing the infrastructure to run them reliably is the problem I spent the summer solving.

---

## The first thing we needed: a cleanup guarantee

When you are running multiple real processes in a test, the most dangerous failure mode is processes that outlive the test. If a test crashes midway and leaves a Bitcoin node running in the background, the next test starts and finds ports already in use, or worse, connects to the wrong process without knowing it. That kind of interference between tests produces failures that are nearly impossible to diagnose.

The solution is a cleanup manager: a component whose only job is to know about every process and resource the test created, and to shut them all down when the test ends, no matter how it ends. Pass, fail, exception, Ctrl+C, all of it. We built this as a LIFO stack. You register resources in the order they were created, and they are torn down in reverse, because you cannot safely stop a process that depends on another process that is already stopped.

The tricky part was signal handling. When a test is killed mid-run by a SIGTERM or SIGINT, the process needs to run cleanup before it exits. The natural instinct is to install signal handlers inside the component's constructor. But that creates a problem: any code that creates a cleanup manager for testing the manager itself, not for running a real test, also accidentally installs global signal handlers as a side effect. That pollutes the test environment.

We solved this by separating the two concerns. The cleanup manager's constructor does nothing to signals. There is a separate, explicit call to install the handlers, and the framework base class makes that call at the right moment. A standalone unit test for the cleanup logic never touches global signal state at all.

---

## Port isolation

Running tests concurrently, which the runner supports via a `--jobs` flag, means two test runs cannot use the same port at the same time. The obvious approach is to assign random ports. The problem with random ports is that randomness introduces flakiness: sometimes the port is in use by something else on the machine, and the failure looks like a Braidpool bug.

The approach we took is deterministic port bands. Before any test starts, the runner assigns each test a numeric seed. Each seed maps to a fixed range of 100 port numbers, starting from a base that sits well below the kernel's ephemeral range. A test with seed 0 gets ports 16000 through 16099. A test with seed 1 gets 16100 through 16199. Within each band, ports are subdivided by component type, Bitcoin RPC, Bitcoin P2P, Braidpool P2P, Braidpool RPC, Stratum, so two different components in the same test cannot accidentally share a port either.

There is one defensive check that turned out to be not-paranoia at all. On some Linux systems, the kernel's ephemeral port range, the ports the OS uses for outgoing connections, starts at 16000 or even lower. If our test band overlapped that range, the OS could silently assign one of our test ports to some unrelated outgoing connection, and the resulting address-in-use error during node startup would look exactly like a bug in Braidpool. We read the kernel's actual ephemeral range from a proc file at startup and refuse to proceed if there is an overlap.

---

## The IPC socket problem

Bitcoin Core and the Braidpool node communicate over a UNIX domain socket. Braidpool connects to Bitcoin Core's mining interface over this socket to get block templates and submit solved blocks. In the test setup, Bitcoin Core opens this socket at a path we configure.

The problem: we initially used a fixed socket path. When we ran two tests concurrently, the second Braidpool process would connect to the first test's Bitcoin Core instance, because the socket path was the same. The tests would interfere with each other in ways that were extremely confusing to debug.

The fix was to give each test its own socket path inside its isolated temporary directory. Since every test run gets its own directory, and directories are named by port seed, there is no collision. It sounds obvious in retrospect, but this is the kind of thing that only reveals itself when you actually run two tests at the same time.

---

## The base class and why it needed a metaclass

Once we had a working cleanup manager, port allocator, and node launcher, the question became: how do you build a framework that future developers can use without understanding all of this machinery?

The answer is a base class. Every functional test script subclasses it and implements exactly two methods: one to declare what the test network should look like, how many nodes, how many miners, how many initial blocks, and one to actually run the test. The framework base class handles everything else: parsing arguments, creating the test directory, starting processes, connecting the node handles, running cleanup, writing timing reports. A test author should be able to write a useful test in about fifteen lines.

Enforcing this contract, making sure subclasses actually implement those two methods and do not accidentally override parts of the lifecycle they should not touch, required a metaclass. A metaclass in Python runs when the class is defined, not when an instance is created. So if a developer forgets to implement the required method, they get a clear error the moment they import the test file, before anything runs. If they accidentally override the lifecycle method that orchestrates the whole test, they get a clear error at the same moment. This felt like the right tradeoff: a small amount of unusual Python in the framework code, in exchange for very clear, early feedback for everyone who uses it.

---

## Reporting, including the crash case

Every test run produces a JSON report with a start time, end time, duration, and status. This sounds simple. The interesting part is what happens when a test subprocess crashes before it can write its own report.

The runner creates a fallback report for each test at the moment the subprocess is launched. If the subprocess finishes normally, the subprocess writes its own report and the fallback is never used. If the subprocess crashes, whether from a segfault in the Braidpool node, an OOM kill, or the runner's own timeout, the fallback report is finalized by the runner instead. This guarantees that every test always produces a machine-readable result, no matter how it dies.

The report file itself is written atomically. We write to a temporary path first, then rename it to the final path. On Linux, rename is atomic at the filesystem level, so a reader scanning the results directory never sees a half-written file.

---

## What I learned

I came into this project thinking of testing infrastructure as unglamorous support work. What I found was a set of systems problems that required careful thought about process lifecycle, concurrency, signal semantics, port management, and failure modes. Every design decision was driven by a real failure we had hit or a failure we could clearly see coming.

The framework is now in review across two pull requests: [#491](https://github.com/braidpool/braidpool/pull/491) and [#526](https://github.com/braidpool/braidpool/pull/526). THe next step is writing rigorous integration tests for the repositry so we try to ensure no bugs are leaked to production.
