---
layout: post
title: "How I'm contributing to Stratum V2"
date: 2023-07-06
author: Lorenzo Bottelli
categories: ['Stories', 'Mining']
---

Share

> **Introduction**

As a Summer of Bitcoin 2023 student, I have the opportunity to contribute to
Stratum V2, an open-source project that will have a huge impact on Bitcoin
mining, enhancing efficiency and security for miners and the overall network.

In the [previous article](/@lobarrel/how-stratum-v2-is-setting-a-new-standard-
for-pooled-mining-22a6de5fe1ed) I explained some of the main features that
Stratum V2 brings to the table and how it represents a big step forward
compared to the current Stratum V1 protocol.

> **Stratum V2 Roles**

In order to understand my contribution to the project, a high-level
explanation of the different actors involved in Stratum V2 and how they
interact is necessary. There are three main actors that interact through the
Stratum V2 protocol: one or more Mining Devices, a Proxy and a Pool.

A Proxy is basically a server which takes multiple downstream connections from
Mining Devices and opens a single upstream connection to the Pool. This
happens through different types of channels introduced by Stratum V2, which I
won’t elaborate on as it’s not the focus of this article.

Mining Devices can run the new SV2 firmware or the old SV1 firmware. If a
Mining Device is running the SV2 firmware it can connect to a SV2 Proxy which
is itself connected to a SV2 Pool. If a Mining Device is running the SV1
firmware it can’t connect to a SV2 Proxy, but it will connect to a Translation
Proxy. A SV1>SV2 Translation Proxy is just like a normal Proxy but it has an
additional translation logic which allows it to translate downstream SV1
messages sent by SV1 Mining Devices into upstream SV2 messages directed to the
SV2 Pool and vice versa.

Here’s a visual representation of the two main configurations discussed above,
depending on which firmware is ran by the Mining Devices.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*MCHjqRe86vle0h50TRIw-Q.png"/>
</figure>

Configuration 1

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dmbNkD5D-u45r44go_cf0g.png"/>
</figure>

Configuration 2

> **The message-generator**

The message-generator is a tool that allows to test the different roles by
generating and sending messages to the tested role and by checking the
responses received. Currently the message-generator can only generate and
handle SV2 messages, but in order to test a configuration with a Translation
Proxy the message-generator has to support SV1 messages as well. This is the
focus of my contribution to Stratum V2.

While SV2 messages are transmitted as binary, SV1 messages are transmitted as
JSON-RPC which obviously adds a substantial overhead making it less efficient
as I mentioned in the previous article. This, however, makes my life a little
easier because the tests executed by the message-generator are written in JSON
and follow a certain test format regardless of which types of messages are
sent.

If you are curious to learn more here’s a couple resources that I produced:

  * A list of the main SV1 messages, their structure and JSON-RPC examples: <https://gist.github.com/lobarrel/359cdf505cc21acd853f55077768d989>
  * An in-depth code description of the message-generator: <https://gist.github.com/lobarrel/a8dc88112038c1eb77099a6116b75d47>


