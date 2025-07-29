---
layout: post
title: "Summer of Bitcoin 2025 — Lightning Network Daemon"
date: 2025-06-07
author: Nishant Bansal
categories: [LND, Stories]
---

## Title:

Continuous Fuzzing for LND

## Technologies:

Go, Docker, AWS S3, GitHub, Kubernetes

## **Problem Statement:**

Modern Go projects lack a robust, low-maintenance solution for continuous
fuzzing. While ClusterFuzzLite has been the go-to tool, its declining
maintenance and reliance on LibFuzzer (instead of Go’s native fuzzing engine)
create friction for developers. This forces teams to manually manage fuzz
targets, execute tests, and maintain corpora, leading to outdated tests that
fail to catch bugs as code evolves.

## **Proposed Solution:**

This project aims to replace ClusterFuzzLite with a dedicated fuzzing
framework for Go that leverages the language’s native fuzzing capabilities (go
test -fuzz). Unlike ClusterFuzzLite, the solution will natively integrate with
Go toolchains, automate execution via built-in coordination logic, and persist
corpora between runs. By eliminating dependencies on third-party engines like
LibFuzzer, the framework will reduce setup complexity and ensure compatibility
with Go’s evolving standards.

## **Project Goals**

  1. Develop a continuous fuzzing tool that triggers automated tests on code changes or at scheduled intervals.
  2. Periodically synchronise with Go project’s GitHub repository.
  3. Automatically detect and execute available fuzz targets.
  4. Implement coordination logic to run the fuzzing service 24/7 as a long-running cloud-based service.
  5. Persist the corpus in cloud storage (AWS S3) to retain and evolve seed inputs over time.
  6. Detect when the fuzzer uncovers a bug and automatically generate an issue report in a private GitHub repository.
  7. Provide documentation for developers to adopt the tool and maintain fuzz targets with minimal effort.

# **Tracking Progress**

I will publish a Medium blog to document every aspect of this project. The
blog will include:

  * **Weekly Progress Reports:** Every Sunday, I will post a summary of completed tasks, encountered challenges, and plans for the next week.
  * **Pull Request (PR) Links:** As I submit code for review, I will link each PR in the blog so that mentors and community members can easily track changes.

Feel free to bookmark and follow the blog.

# **Acknowledgements**

I would like to express my heartfelt gratitude to my mentor Matt Morehouse. He
has been incredibly supportive and motivating during the application phase.
Whenever I needed help, he guided me toward solutions. Our discussions
throughout the application phase were fascinating and pushed me to expand my
learning.

A big thank you also goes out to the LND community for their unwavering
support.

Finally, I’m grateful to the Summer of Bitcoin team for organising Summer of
Bitcoin, which has made a tremendous positive impact on both the open-source
community, the Bitcoin ecosystem, and me.

# **Conclusion**

Being selected for Summer of Bitcoin 2025 under the LND organisation is a
tremendous honour. From my first steps as an LFX mentee working on checksum
verification to now working on continuous fuzzing for LND — both security-
oriented projects — this journey has been both challenging and rewarding. Over
the next three months, I look forward to working closely with my mentors,
collaborating with fellow Lightning contributors, and delivering a robust,
user-friendly continuous fuzzing solution that strengthens both the Bitcoin
and Lightning Network ecosystems. If you’re interested in open-source Bitcoin
development, don’t hesitate — start by contributing a small issue fix or
improving documentation. You never know where it might lead!

— Nishant Bansal

