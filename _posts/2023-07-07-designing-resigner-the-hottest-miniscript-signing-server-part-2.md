---
layout: post
title: "Designing Resigner: The hottest miniscript signing server. Part 2"
date: 2023-07-07
author: Bryan Elee
categories: ['Stories', 'Wallets']
---

I was fortunate enough to be accepted to the Summer of Bitcoin program for
2023 under Ledger and I’m responsible for implementing this project General
purpose hot signing service for miniscript policies (can be found here).

There’s not really an accepted approach on how to implement a general signing
service — possibly because the are no such service existing at this point in
time. Some design decisions we made to enable some core features to be
completed within three months, Since this is a summer of bitcoin project that
has to show results within three months — using python rather than rust, while
some other decisions were made to reduce dependencies and reflect our target
audience — Bitcoin developers — for example using sqlite3 database module in
python rather than a high performance database such a postgres.

A decision was made was made to forgo policies/conditions that doesn’t further
“our” cyberphunk aspirations, such as enforcing the “compliance with
regulatory environment”.

**The Resigner Output Descriptor**

Resigner is a general signing server hence it supports any descriptors that
follows the simple restrictions defined below. On close inspection of the
descriptors previous part, we can see that each descriptor requires the
minimum of the following to satisfy the miniscript

  * The server’s signature and a single users signature
  * and a relative lock that allows the UTXO to be spent after a certain time, blocks have elapsed since the creation of the UTXO with just the users keys.

Hence every satisfaction to the miniscript, while the time specified in the
relative lock has not elapsed must include the servers signature. There are
other requirements such as:

  * At least one of the keys in the descriptor should be a wildcard, to allow for new addresses to created deterministically. The xpub corresponding to Resigner’s key should be a wildcard key in the form [84h/0h/0h]xpub/0/* or xpub/0/* (where previous steps are already hardened), so that every participant can generate a new address irrespective of resigner and other participants.

Just about any output descriptor conforming to the above can be used with
resigner.

**Relative timelocks and Recovery paths**

Another point to note is that after the relative lock has elapsed, all the
utxos will have to be spent to an address created from a similar locking
script, in other to keep using the signing service. Time starts ticking when
you receive a payment. That is if you want the recovery path to never be
available, each coin must be spent at least once every `N` blocks/secs. (With
`N` the configured value of the timelock.)

**Bitcoind**

Resigner connects to the bitcoin network through a bitcoin-core node, to
access the blockchain in order to monitor transactions, as might be required
to enforce the spending conditions. Resigner also leverages bitcoin-core’s
wallet psbt capabilities for signing PSBTs. As a result of this the minimum
supported version of bitcoin core is 25.0

**REST API**

Resigner exposes a single post endpoint:

    
    
    POST /process-psbt

Signs a PSBT using keys held by resigner

Request body:

    
    
      
    Content-type: "application/json"  
      
    `example`  
    {"psbt": ""}

Response

    
    
      
    200 OK  
    Content-type: "application/json"  
      
    {"psbt":"", "signed": boolean, "complete": boolean}

**Configuration**

The configuration file is used by resigner is in the toml file format and
consists of option=value entries, one per line. The spending policies and
server configs options can be specified in a single toml file.

More information regarding configuring resigner can be found here and setting
up spending policies can be found here.

**Current State of “Resigner” development**

The goal of having a standalone signing server that exposes a REST endpoint to
receive a PSBT and signs it (or not) according to the preset spending
conditions, has been achieved. There are tests and docs for anyone savvy
enough to start playing with resigner.

The challenges we face in the development will hopefully inform the design the
resigner. We hope that resigner kick starts the development of new open source
signing services and reinforce the development of miniscript. In the near
future we hope to add support for taproot descriptors and muSig2.

Special thanks to my mentor Salvatore Ingala for his unwavier support and all
the miniscript lovers on X(formerly twitter.com) for cheering me on, most
especially @Rob1Ham.

PS. I have avoided much jargon and complexity so it remains accessible to all
that want to play with it. A more indepth guide into resigner and it’s
development can be found here

If you happen to test out resigner, I would happy to have your feedback here
@Rxbryan6
