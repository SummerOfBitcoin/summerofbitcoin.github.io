---
layout: post
title: "My contribution to Stratum V2 as a Summer of Bitcoin student"
date: 2023-06-13
author: Lorenzo Bottelli
categories: ['Stratum', 'Mining', 'Stories']
---

> **Introduction**

In this article I want to share my experience as a Summer of Bitcoin 2023
student, explaining what I have been working on and providing some tips that
might be helpful for future students.

Summer of Bitcoin is an internship program that gives you the opportunity to
work on a real Bitcoin-related open-source project. Among the many projects
available I chose to apply to work on Stratum V2. I gave a general overview of
what Stratum V2 is and how it represents a big change in the Bitcoin mining
industry in a previous article: https://medium.com/@lobarrel/how-
stratum-v2-is-setting-a-new-standard-for-pooled-mining-22a6de5fe1ed.

The reasons I was so interested in working on Stratum V2 are mainly two:

  * I find mining one of the most fascinating aspects of the Bitcoin technology, being the beating heart of the entire ecosystem. The growth of a new standard for pooled mining can impact the mining industry as a whole, enhancing security, decentralization and efficiency.
  * The programming language used in Stratum V2 is Rust, which I already had some experience with. Rust is one of the fastest-growing programming languages right now and is rising in popularity also thanks to big tech companies adopting it. Furthermore, Rust has become the second language officially accepted for Linux kernel development (along with C), which demonstrates how robust and reliable it is.

While I was waiting for my proposal to be evaluated, I started exploring the
Stratum V2 Github repo to get familiar with the code structure and the
libraries. Having a clear idea of how the code is organized is really
important, especially when working on projects with many different modules and
sub-projects.

I finally got accepted and after getting in contact with my mentor I was ready
to start.

> **My contribution**

The goal of my contribution was to integrate Stratum V1 capabilities inside
the message-generator. The message-generator is basically a testing tool used
to test the interoperability of different roles, which I explained more in
detail in a previous article: https://medium.com/@lobarrel/how-im-
contributing-to-stratum-v2-c6b24b6e9cad. Essentially what the message-
generator does is run the tested Stratum V2 role (for example a pool or a
proxy), generate and send messages to the tested role and check that the
expected responses are received. The message-generator only supported Stratum
V2 messages, so it was not possible to properly test the Translator Proxy. The
Translator Proxy, as explained in the previous article, sits in between a
mining device running Stratum V1 firmware and a Stratum V2 pool, and is
responsible for translating the SV1 messages it receives from downstream into
SV2 messages to send upstream (and vice-versa). So in order to properly test
the Translator Proxy it was necessary to add support for Stratum V1 messages
in the message-generator.

I spent the first few weeks doing some study, research and planning. I started
by doing some research on the Stratum V1 protocol and I collected the main SV1
messages in a document highlighting their structure and format and providing
some examples. Then I went through the existing code of the message-generator
and tried to understand every single step of code execution from start to
finish. To help me with that I produced a really detailed code description
(can be found here) in which I explain every function, struct and step of the
execution process. Doing this has really helped me to get a deep understanding
of the code I was going to work on, and it can also be helpful for future
developers who want to work on it. After that, I had a more precise idea on
how to implement SV1 messages and integrate them in the existing code, so I
wrote down a draft document of the main things to do in order to accomplish
that. Then I was ready to start playing with the code.

I started by going through the _sv1_ module where all the SV1 messages are
defined and I fixed some bugs that prevented it from compiling since I needed
to use that module inside the message-generator.

The tests used by the message-generator are JSON files with a precise
structure containing a list of messages to be sent, a list of actions to be
completed (such as checking a particular response), a list of commands to be
executed (such as running a pool or a translator proxy) and some information
about the connection details of the tested role. Unlike SV2 messages that are
sent as binary, SV1 messages are sent as JSON objects following the JSON-RPC
protocol. So the messages contained inside the JSON test can be directly
parsed into JSON-RPC requests and sent to the tested role. After implementing
the parsing of SV1 messages it was time to deal with actions. The things that
the message-generator can do after receiving a response are: verify that the
response ID corresponds to the ID of the request that has been sent and
compare the value of a certain field of the response with the expected value.
So I implemented the parsing of these SV1 actions and added the logic to check
the action results inside the Executor, the component that manages the test
execution process. Then, after dealing with some problems related to the
communication between the message-generator and the translation proxy, I wrote
an example of an executable JSON test and it passed. This test can be used as
a reference to write more tests and check that everything is ok.

Now the message-generator can successfully handle SV1 messages, allowing to
test the interoperability of SV1 software with SV2 software, such as a SV1
mining device with a Translator Proxy connected to a SV2 pool. This is a
relevant feature especially in the transition phase from Stratum V1 to Stratum
V2, where miners are still running SV1 software on their machines but want to
explore and experiment the benefits of Stratum V2.

Here you can find my PRs:

## defined sv1 messages in the MG + added serde for some v1 newtypes by
lobarrel · Pull Request #602 ·…

### added sv1 messages definition inside the mg, fixed some bugs in v1/utils
and implemented examples of serialization and…

github.com

## Sv1 integration in MG by lobarrel · Pull Request #606 · stratum-
mining/stratum

### added test version to MG tests. Added parsing of sv1 tests (sv1 messages,
sv1 actions, sv1 action results) and…

github.com

> **Conclusion**

Participating to the Summer of Bitcoin internship was by far one of the best
decisions I have made. Having the opportunity to actively contribute to a
growing and exciting Bitcoin project is not something usual, so it makes the
experience even more valuable. From a technical point of view I consolidated
my previous knowledge and I learned much more, especially thanks to my mentor
and the other developers I worked with. The thing that I find most valuable is
that nobody has told me exactly what to do, so I had to study, research and
figure it out by myself (of course knowing that there were more experienced
people ready to help me) and I really liked this type of challenge as it
particularly stimulated my problem solving skills.

If you are interested in the Bitcoin technology and have strong motivation I
highly recommend you to look into the Summer of Bitcoin as an opportunity to
expand your knowledge and put yourself to the test.
