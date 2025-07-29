---
layout: post
title: "Building the Send Functionality for BIP21 URIs in LDK-Node: A Technical Deep Dive"
date: 2024-08-13
author: "Ian Slane"
categories: [Stories, LDK]
---

This article discusses implementing send functionality for BIP21 URIs in LDK Node, covering on-chain transactions, BOLT11 invoices, and BOLT12 offers.

### Key Updates from Previous Post

*   **Error Handling:** The `receive` method was improved to return specific errors (`Error::OfferCreationFailed` or `Error::InvoiceCreationFailed`) instead of silently logging them.
*   **URI Formatter:** The helper function for formatting the URI for QR codes was enhanced. Now named `format_uri`, it capitalizes the values for both `lightning=` and `lno=` keys to improve scannability and support for new parameters.

### Implementing the Send Functionality

The `send` method is designed to parse a BIP21 URI and attempt payments in a specific order: BOLT12 offer, then BOLT11 invoice, and finally an on-chain transaction as a fallback.

**Method Breakdown:**

1.  **Parse URI:** The input string is parsed into a `bip21::Uri`. An `Error::InvalidUri` is returned on failure.
2.  **Network Validation:** It checks if the URI's network matches the user's configured network, returning `Error::InvalidNetwork` if they don't align.
3.  **BOLT12 Offer Handling:**
    *   If a BOLT12 offer is present, it attempts to send the payment.
    *   On success, it returns a `QrPaymentResult::Bolt12` with the `payment_id`.
    *   On failure, it logs the error and proceeds to the BOLT11 invoice. BOLT12 is prioritized due to its advanced features.
4.  **BOLT11 Invoice Handling:**
    *   If an invoice is present (and the BOLT12 payment failed or wasn't present), it attempts to send the payment.
    *   On success, it returns a `QrPaymentResult::Bolt11` with the `payment_id`.
    *   On failure, it logs the error and falls back to an on-chain transaction.
5.  **On-chain Transaction Handling:**
    *   As a final fallback, it checks for a payment amount in the URI. If no amount is specified, it returns `Error::InvalidAmount`.
    *   It sends an on-chain payment to the address in the URI.
    *   On success, it returns a `QrPaymentResult::Onchain` with the transaction ID (`txid`).

### Debugging BOLT12

A bug was discovered during integration testing for BOLT12 payments. The system would return a payment ID even if the payment failed (e.g., due to insufficient channel liquidity), preventing the fallback mechanism from triggering. The payment would get "stuck" at the offer stage.

A fix was proposed in LDK to abort the payment early if the available liquidity is less than the required amount, ensuring the fallback to other payment methods works correctly.

### What's Next

The author plans to discuss the integration tests and the challenges involved in more detail in a future post.
