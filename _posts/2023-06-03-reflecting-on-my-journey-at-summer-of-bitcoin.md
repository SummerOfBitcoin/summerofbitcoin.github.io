---
layout: post
title: "Reflecting on My Journey at Summer of Bitcoin"
date: 2023-06-03
author: Tarek Elsayed
categories: ['Stories', 'Core Lightning']
---

# Introduction

Greetings everyone! My name is Tarek Elsayed, and I am a third-year computer
engineering student from Egypt. I am thrilled to share my incredible
experience as an intern at Summer of Bitcoin, a global online summer
internship program focused on introducing university students to Bitcoin open-
source development. Throughout this blog post, I will take you on a journey
through my time at Summer of Bitcoin, specifically my work at CLN Toolkit’s
coffee, a plugin manager for Core Lightning. Let’s dive in!

# The Project and My Role

My primary focus during the Summer of Bitcoin was on CLN Toolkit’s coffee, the
plugin manager for Core Lightning. coffee handles all the configuration and
installation of plugins for your Core Lightning node. I was entrusted with the
responsibility of adding new commands to the command-line interface (CLI) and
designing an interface to enable web applications or other platforms to run
coffee as a server. Additionally, I actively addressed various mid-way issues
that arose, contributing to the overall improvement and stability of the
project.

# Pre-Summer of Bitcoin Readiness:

Before the official start of Summer of Bitcoin, I had some excellent
preparation. Vincenzo had set up a bunch of tasks to make it easier for us to
begin with the project. These tasks were incredibly helpful, especially since
I wasn’t very familiar with Rust at that time. With Vincenzo’s guidance, these
early challenges helped me get comfortable with Rust and allowed me to start
making contributions through pull requests from the beginning. This thoughtful
approach laid the foundation for my successful journey with coffee and Summer
of Bitcoin.

# Milestones Achieved

Since the program commenced on May 15th, I have consistently driven my project
towards its successful completion. Here is an overview of the significant
milestones and accomplishments that have marked this journey:

  * **Support for Multiple Commands** : I added support for various commands in the CLI, allowing users to perform various actions. I successfully integrated several new commands into the CLI. These include commands such as `list` for displaying installed plugins, `list remotes` (later replaced by `remote list`) for showcasing remote repository plugins, `remove remote` for eliminating remote repositories, `show` for accessing plugin README file, `remove` for uninstalling plugins, and the `upgrade` command.
  * **Implementing Endpoints to the HTTPD Crate** : To enhance coffee’s functionality, I implemented endpoints that provide access to a wide array of coffee CLI commands through the HTTPD crate.

These additions to the HTTPD crate allow users to interact with coffee through
HTTP requests. This integration opens up exciting possibilities for
integrating coffee into various web applications, expanding its accessibility
and usability. By providing endpoints for these commands, coffee becomes more
versatile and empowers developers to incorporate its functionality seamlessly
into their own projects.

> You can find all my PRs and open issues here

## Meeting Vincenzo: My Mentor

From the very beginning, I was fortunate to be paired with Vincenzo, who has
been an incredible mentor throughout this internship. Vincenzo’s guidance and
responsiveness have played a significant role in my progress and understanding
of the project. Having a supportive mentor is crucial, and I am grateful to
have worked alongside someone as experienced and helpful as Vincenzo.

## Contributing to the Lightning Ecosystem:

My time with coffee and Summer of Bitcoin has made me more interested in
contributing to the broader Lightning Network projects and specifications.
Exploring the ins and outs of the Lightning Network and its protocol has
sparked my curiosity to be a part of it.

Notably, my recent work has given me some hints about how we can improve the
larger Core Lightning project. I've been focusing on an issue where Core
Lightning encounters problems when there are missing imports. I am determined
to investigate this issue in depth. My plan involves not only opening an issue
on GitHub but also exploring the possibility of creating a patch if feasible.

# Challenges and Growth

My journey during the Summer of Bitcoin has been full of rewards and learning
experiences, along with a fair share of challenges. While working on various
coding tasks and fixing issues, I encountered moments that took me out of my
comfort zone. These challenges have helped me grow both personally and
professionally. Notable achievements include:

**_Embracing Rust_** _:_ As I continue to learn Rust, the programming language
used for coffee’s development, I’ve gained a better understanding and feel
more confident using it. This has expanded my skills and opened doors for
future projects.

**_Insights into Bitcoin Development:_** My involvement with coffee has given
me valuable insights into Bitcoin and how software development works. I’ve
come to appreciate the importance of security, the decentralized nature of
projects, and how the open-source community collaborates to create impactful
software.

These successes despite challenges showcase my Summer of Bitcoin experience,
where every difficulty has turned into a way for me to learn more and become
more connected with the world of Bitcoin open-source development.

# Conclusion

My time at Summer of Bitcoin has been amazing. Working on CLN Toolkit’s coffee
and contributing to the Bitcoin open-source development community has been a
valuable experience for me. I’m thankful to have had Vincenzo as my mentor,
helping me through the project and creating a friendly environment.

If anyone is interested in Bitcoin development, I can wholeheartedly recommend
Summer of Bitcoin as a great opportunity to learn and grow.
