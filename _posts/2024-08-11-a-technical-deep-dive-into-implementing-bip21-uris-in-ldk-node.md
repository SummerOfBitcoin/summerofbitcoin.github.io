---
layout: post
title: "A Technical Deep Dive into Implementing BIP21 URIs in LDK-Node"
date: 2024-08-11
author: "Ian Slane"
categories: [Stories, LDK]
---

In a previous article, the author explored LDK Node and the concepts of BIP21 QR code support, which allows users to request bitcoin payments using a single QR code for both on-chain and lightning payments. This article focuses on the practical implementation, specifically the integration of the `bip21` crate into the `unified_qr.rs` payment handler.

The `bip21` Rust crate is a library for handling BIP21 bitcoin payment URIs, chosen for its performance, flexibility, and compliance with the BIP21 standard. To integrate it, a custom URI type called `LnURI` was defined to include extra parameters for the URI, such as optional BOLT11 invoices and BOLT12 offers. The `NetworkChecked` type marker is used to ensure the address's network is validated during parsing.

To handle the custom parameters, the `SerializeParams` and `DeserializeParams` traits were implemented for an `Extras` struct. The `SerializeParams` trait converts the `Extras` struct into a format that can be included in the URI, while the `DeserializeParams` trait converts the serialized data back into the `Extras` struct. The author notes that currently, the project uses a single "lightning" key for both BOLT11 invoices and BOLT12 offers, but plans to use the correct "lno" key for BOLT12 offers in a future commit.

A complete URI with a BOLT11 invoice and a BOLT12 offer could look like this:
`BITCOIN:BC1QUM5RL5AAMKC5YEHL9H9SRXTR4NCDEGEUVSFUGS?amount=1&message=LDK%20Rocks&lightning=LNBC1000M1PN8GSKYDQA2PSHJMT9DE6ZQ6TWYP3XJARRDA5KUNP4QVMG8RY0YLRXECA7YE5V50MM9JTSXWW4FJNNLCU678KU0LH2SVYJQPP5DGVG4Y909VMUUJ9J92KMYG55AVZQ8P6P4CQ4NP0RF0MG67HQ4MZQSP5WQM2JFS0H43575DT9J9WQM4SW82XGKLE6LQGFX6744JZQ0RXA9QQ9QYYSGQCQPCXQRRAQ3T0D5UHM570YGP6DSPG37VG9FNSQ200DHQZU4L256WMSYK3L4H2YXV23P9SE3ZQLJD5FYCENC57L8SN3PQKW3AMLCSNGWNETL8ZH2WGQT80HPV&lno=LNO1PGXXVET9VSSX6EFQWDSHGUCKYYP68KS6WHN8SSETRPXPDC3PWLR7H0JK07H5Z0Z8079JYXS0DGKQCLG`

This implementation is integrated into the `unified_qr.rs` payment handler, which enables the generation and parsing of these URIs. The author plans to discuss the `receive` and `send` methods, as well as the logic behind making a payment given a BIP21 URI, in a future post.
