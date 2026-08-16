---
layout: post
title: "Breaking Lightning (Constructively): My SoB Mid-Term Reflections"
date: 2026-06-27
author: Chandra Pratap
categories: [Development, Open-Source, Security, Lightning-Network]
image: ../assets/images/blog_content/Smite.png
---

## Who I am and what project I am working on

I am Chandra Pratap (aka Chand), a final-year Mathematics student at NIT Surat in India. For Summer of Bitcoin 2026, I am working on the project "_Enhancing Smite: Advanced IR Mutation and Effectiveness Evaluation._"

I was also a Summer of Bitcoin intern in 2025 with the Lightning Network organization, where I focused on [improving the fuzzing suite for Core Lightning](https://chand-ra.github.io/sob/2025-08-17-sob-final-report/). In many ways, my current work on [Smite](https://github.com/morehouse/smite/) is an advanced continuation of that original project.

## What problem my project is solving

### Fuzzing imperative in the Lightning Network

[Fuzz testing](https://en.wikipedia.org/wiki/Fuzzing) is an automated software testing technique that feeds random inputs into a program to uncover hidden vulnerabilities. In the context of the Lightning Network, fuzz testing is particularly crucial because real financial transactions are at stake. Past fuzz tests have identified critical vulnerabilities, such as [invoice parsing bugs in CLN](https://morehouse.github.io/lightning/cln-invoice-parsing/) and the [LND "onion bomb"](https://morehouse.github.io/lightning/lnd-onion-bomb/), which could have easily caused widespread node crashes across the network.

### Limitations of naïve fuzzing

Standard, byte-level fuzzing struggles to reach deep logic paths in complex, stateful protocols. To achieve deeper coverage, a fuzzer must be "structure-aware," enabling it to generate and mutate sequences of valid messages that can actually survive initial parsing checks. This is difficult with naïve fuzzing because protocol messages carry strict state dependencies. For example, a `node_announcement` gossip message will be immediately rejected unless it contains a valid `node_id` derived from a preceding `channel_announcement`.

### Smite’s IR architecture

Much like [Fuzzamoto](https://github.com/oss-garage/fuzzamoto) does for Bitcoin full-node implementations, Smite provides a [coverage-guided fuzzing](https://google.github.io/clusterfuzz/reference/coverage-guided-vs-blackbox/) framework specifically designed for the Lightning Network. Smite solves the dependency problem by utilizing a custom [Intermediate Representation (IR)](https://en.wikipedia.org/wiki/Intermediate_representation). This IR captures the necessary type and structural knowledge of the protocol, allowing the fuzzer to generate "short programs" that are executed within a Virtual Machine.

Once a valid IR program is generated, the fuzzer must mutate it to explore new logic paths. This generally happens in two ways:

1. **Byte-Level Mutation:** The IR is serialized into raw bytes, and traditional bit-flipping occurs (this method is not used in Smite).

2. **Structural IR Mutation:** The blueprint of the program itself is mutated before serialization. This is required to restructure the program or extend it with entirely new protocol flows.

### My contribution: Advanced mutation and rigorous evaluation

While Smite’s IR solves the initial dependency problem, generating a valid program is only half the battle. To discover deep vulnerabilities, the fuzzer must be able to aggressively mutate those blueprints to explore new logic paths without breaking the underlying protocol rules.

This is exactly where my project comes in, solving two specific problems for Smite:

1. **Expanding State Exploration:** I am building four advanced structural mutators (`InstructionDelete`, `InstructionReorder`, `GeneratorInsertion`, and `SpliceMutator`). Instead of just tweaking data fields, these mutators actively alter the flow of the IR programs. Think dropping messages, reordering operations, and injecting entirely new protocol flows. This allows the fuzzer to explore vastly more complex network states while dynamically respecting the Lightning Network's constraints.

2. **Proving Fuzzer Efficacy:** How do we actually know if a new mutator improves the fuzzer or just wastes compute time? Previously, we couldn't easily tell. To solve this, I designed and implemented an end-to-end [**Effectiveness Evaluation Framework**](https://github.com/morehouse/smite/issues/115). Using historical vulnerabilities as ground truth, the framework orchestrates massive fuzzing trials and uses rigorous statistical tools (like Mann-Whitney U tests, Vargha-Delaney A12 effect sizes, and Holm-Bonferroni corrections) to provide statistical evidence about whether a new configuration actually improves our bug-finding capabilities.

## What I completed in the first six weeks

I have successfully implemented three out of the four advanced structural mutators I proposed: [`InstructionDelete`](https://github.com/morehouse/smite/pull/60), [`InstructionReorder`](https://github.com/morehouse/smite/pull/61), and [`GeneratorInsertion`](https://github.com/morehouse/smite/pull/126). This leaves only the `SpliceMutator`, which I expect to finish in a couple of weeks.

Alongside the mutators, I built the statistical evaluation framework from the ground up to measure their [effectiveness](https://github.com/morehouse/smite/issues/115). I designed this framework to be robust enough to evaluate any new feature that might extend Smite's capabilities in the future, and I wrote a Python script to fully automate the [evaluation pipeline](https://github.com/morehouse/smite/pull/125).

Additionally, a significant portion of my time was spent resolving a blocking architectural problem known as the **[implicit IR dependency issue](https://github.com/morehouse/smite/issues/75)**. More on this in the following section.

## The hardest problem I faced

The hardest problem I faced was undoubtedly the [implicit IR dependency issue](https://github.com/morehouse/smite/issues/75). In Smite's Intermediate Representation, certain instructions implicitly depended on the state of others, which caused our advanced mutators to inadvertently generate semantically invalid programs.

Resolving this took weeks of brainstorming, architectural back-and-forth, and implementation rewrites, ultimately forcing us to redesign the mutators using the existing input-output dependency graphs to respect affine state rules by construction. It was a massive hurdle, but getting it [solved](https://github.com/morehouse/smite/pull/97) was incredibly rewarding.

## What I learned about Bitcoin, open source, or software/design

I’ve learned just how vast the field of fuzzing research really is. Such a fundamentally simple concept, _throwing random inputs at a program_, can be adapted in countless, highly sophisticated ways to find real vulnerabilities across every type of software imaginable.

On a practical level, I also learned a crucial lesson in software architecture: _do the simplest thing first that solves the problem at hand._ It is far too easy to waste time trying to extrapolate an architecture to solve hypothetical future problems that you may never actually encounter.

## What I plan to finish before the final evaluation

My primary goals are to implement the final `SpliceMutator` and to actually run the effectiveness evaluation trials. From there, I will generate detailed reports comparing how the various mutator configurations perform against our current set of targets.

## Links to my work

1. [`InstructionDelete` PR](https://github.com/morehouse/smite/pull/60)
2. [`InstructionReorder` PR](https://github.com/morehouse/smite/pull/61)
3. [`GeneratorInsertion` PR](https://github.com/morehouse/smite/pull/126)
4. [Effectiveness evaluation framework architecture](https://github.com/morehouse/smite/issues/115)
5. [Automated evaluation script PR](https://github.com/morehouse/smite/pull/125)
6. [Implicit IR dependency issue](https://github.com/morehouse/smite/issues/75)
7. [Implicit IR dependency solution PR](https://github.com/morehouse/smite/pull/97)

## Get Involved

Fuzzing the Lightning Network is a massive, ongoing effort, and the ecosystem is always looking for fresh eyes. If you’re interested in protocol security, Rust, or structure-aware fuzzing, check out the [Smite repository](https://github.com/morehouse/smite/) on GitHub. Feel free to spin it up, test it out, and drop your feedback in the issues.

If you want to follow along with my final Summer of Bitcoin progress, you can view my [personal blog](https://chand-ra.github.io/) or [GitHub profile](https://github.com/chand-ra/). You can also reach out to me on [X](https://x.com/chandrapratap37) or [LinkedIn](https://www.linkedin.com/in/chand-ra/) if you have any questions or want to talk about fuzzing!
