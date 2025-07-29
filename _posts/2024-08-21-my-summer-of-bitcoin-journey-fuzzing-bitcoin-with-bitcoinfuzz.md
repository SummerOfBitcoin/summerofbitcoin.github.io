---
layout: post
title: "My Summer of Bitcoin Journey: Fuzzing Bitcoin with BitcoinFuzz"
date: 2024-08-21
author: Kartik Agarwala
categories: [Bitcoin-Fuzz, Stories]
image: ""
---

# Progress Blog Beta - Bitcoinfuzz

August 21, 2024 · 5 min

Table of Contents

  * Introduction
  * Progress
    * Bitcoin Network Protocol Harness
    * RPC Harness
  * Conclusion

# Introduction#

Hello all 👋, welcome back to my blog! As discussed previously, I am working on
a project called bitcoinfuzz and today I wanted to update you all on my
progress as an SoB intern.

# Progress#

Last time we left, I got AFL and honggfuzz to work with bitcoinfuzz. What I
hadn’t disclosed at that time was that in doing so, I got a crash with the
miniscript harness and it was crashing with a _SIGSEGV_ which was weird and
unexpected. I initially thought that something was wrong with the target
itself, but I soon came to realize that it was an actual issue with rust-
miniscript. My mentor and I discussed this and then after a bit of research,
we disclosed the bug to the rust-miniscript team who quickly patched and
upstreamed fixes. We then filed for a CVE and I’m super excited to announce
that this bug has been assigned CVE-2024-44073 with a CVSS Score of `7.5`. If
you are more interested in the bug, you can checkout this blogpost by my
mentor which is more detailed.

One of the major things that I have done after midsem is to implement the
bitcoin network protocol and rpc harnesses which were the original aim of my
proposal whcih I’ll be talking about in this post.

## Bitcoin Network Protocol Harness#

Before I talk about the bitcoin network protocol harness, let us first talk a
_little_ about the protocol itself.

Bitcoin Network Protocol or just _wire_ for short is at the core really
simple. Nodes communicate using a TCP connection and every message consists of
a “header” and an optional “payload” part. Headers are made up of 24 bytes of
data having the following format:

Byte Size| Name| Description  
---|---|---  
4| Start String| Magic bytes indicating network  
12| Command Name| Command Name which specifies message type  
4| Payload Size| Number of bytes in payload  
4| Checksum| DOUBLESHA256(payload)  
  
Payload itself has no specific structure and depends on the network message
type. When payload is empty, we use a fixed checksum value of `0x5df6e0e2`.
The main complexity of this protocol arises from the different network message
types and its deserialization. There are `27` different network message types
that are well documented on the bitcoin wiki.

Talking about the harness, one of the primary challenge was the `checksum`
field in the header, as SHA256 operations are expensive and we want to keep
fuzzing as fast as possible. To overcome this, we need to manually patch btcd
and rust-bitcoin to simply skip the checksum. One of the nice things about
switching to bitcoin core’s `fuzz.cpp` in bitcoinfuzz has been that it has
given us the ability to call a init function once a harness starts executing.
This is particularly useful in this case as we could check and abort here
incase the patches have not been applied. The procedure to do this is really
simple. All you need to do is to create _any_ wire command and send it with
any arbitary value as the checksum, and if there is no error, then patches
have been applied correctly.

Once the checksum is taken care of, the harness works by taking a random
length input from the fuzzer engine, out of which the first byte is used to
select one of the 27 network message type and the remaing bytes are simply
treated as the payload.

## RPC Harness#

RPC (Remote Procedure Call) is a protocol that allows a program to execute a
procedure or function on another computer or network as if it were a local
procedure call. In the context of Bitcoin, RPC is used to interact with and
control Bitcoin nodes and wallets. To use Bitcoin RPC, one typically needs to
run a Bitcoin Core node and configure it to accept RPC connections. Then, it
can interacted with the node using command-line tools, programming libraries,
or custom applications that implement the RPC protocol.

BTCD also has support for RPC calls, and as both bitcoin core and btcd are
bitcoin nodes, they have almost the same RPC calls which perform almost the
same functions. This makes fuzzing RPC functionality an excellent target for
differential-fuzzing.

Initially, I thought that I would need patches once again to hack into btcd’s
rpcserver and I’ll have to make it acessible from my btcd wrapper package
however that turned out to be too tedius with a large patch required. So in
the end what I ended up doing was to have the user launch a btcd server
manually, which would be sent http requests from our rpc harness. I adapted
the excellent pre-existing rpc harness present in bitcoin for this and added
in the functionality for BCTD.

We found multiple bugs using this which we are currently in the process of
disclosing to the BTCD team. 😁

# Conclusion#

To wrap up, with the addition of these two harnesses, I’ve formally completed
my work as an SoB intern. The last few months have been fantastic, and it
cannot be understated how much I have learnt about bitcoin and fuzzing in the
last few months and how good my experience has been. I even ended up getting
my first CVE, so, yay!

I would like end this post by extending my thanks to my mentor Bruno Garcia
who has helped me immensely during this time and the Summer of Bitcoin team
which provided me with this opportunity!

Thank you for reading my blog post! 😁
