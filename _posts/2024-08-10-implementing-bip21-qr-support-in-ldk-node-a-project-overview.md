---
layout: post
title: "Implementing BIP21 QR Support in LDK Node: A Project Overview"
date: 2024-08-10
author: "Ian Slane"
categories: [Stories, LDK]
---

This article discusses the implementation of BIP21 QR code support for LDK Node.

BIP21 is a URI scheme for creating bitcoin payment links or QR codes, which standardizes how bitcoin payments are requested. This is useful because it allows a single QR code to contain both an on-chain bitcoin address and a lightning invoice, simplifying the payment process for users.

LDK Node is a ready-to-go lightning node library built in Rust with the Lightning Development Kit (LDK) and Bitcoin Development Kit (BDK). It aims to be a compact, user-friendly, and modular solution for setting up and managing a lightning node. LDK Node is written in Rust but also provides language bindings for Swift, Kotlin, and Python.

The project's initial setup involved creating a unified QR code builder module. The plan is to create a `UnifiedQrPayment` handler with `send` and `receive` methods. The `receive` method will generate a URI from the user's wallet address, a node-generated invoice, an amount, an optional message, and an expiry time. The `send` method will parse a URI, pay the invoice or offer, and fall back to an on-chain payment if the initial methods fail.

The `bip21` Rust crate will be used for serializing and deserializing the URI. The author plans to discuss the use of this crate and the first draft of the project in future posts.
