---
layout: post
title: "Unit Testing BIP21 URIs in LDK-Node: A Deep Dive"
date: 2024-08-14
author: "Ian Slane"
categories: [Stories, LDK]
---

In my previous post, we talked about the details of implementing the send functionality for BIP21 URIs in LDK Node, exploring the various payment methods including BOLT11 invoices, BOLT12 offers, and on-chain transactions. If you missed it, check it out here! Now, this week, we’ll turn our focus to an equally crucial aspect of the development process: unit testing. Effective unit tests are essential for ensuring the reliability and robustness of our implementation. In this post, I’ll guide you through the unit tests I’ve written to validate the functionality of our BIP21 Unified QR code handler. We’ll cover test cases for each payment type, highlight some of the challenges faced during testing, and examine how these tests help ensure that our payment processing logic functions as expected under various scenarios. Whether you’re working on similar projects or just curious about testing strategies, I hope you’ll find these insights valuable. Let’s dive into the details of how we tested our code and the lessons learned along the way.

## IMPLEMENTING THE UNIT TESTS

Let’s break down the unit tests designed to validate the core functionality of our BIP21 Unified QR code handler. This test covers various scenarios including payment creation, sending, and fallback mechanisms. Here’s a detailed look at what the test looks like/does:

```rust
#[test]
fn unified_qr_send_receive() {
 let (bitcoind, electrsd) = setup_bitcoind_and_electrsd();
 let (node_a, node_b) = setup_two_nodes(&electrsd, false, true, false);

 let address_a = node_a.onchain_payment().new_address().unwrap();
 let premined_sats = 5_000_000;

 premine_and_distribute_funds(
  &bitcoind.client,
  &electrsd.client,
  vec![address_a],
  Amount::from_sat(premined_sats),
 );

 node_a.sync_wallets().unwrap();
 open_channel(&node_a, &node_b, 4_000_000, true, &electrsd);
 generate_blocks_and_wait(&bitcoind.client, &electrsd.client, 6);

 node_a.sync_wallets().unwrap();
 node_b.sync_wallets().unwrap();

 expect_channel_ready_event!(node_a, node_b.node_id());
 expect_channel_ready_event!(node_b, node_a.node_id());

 // Sleep until we broadcast a node announcement.
 while node_b.status().latest_node_announcement_broadcast_timestamp.is_none() {
  std::thread::sleep(std::time::Duration::from_millis(10));
 }

 // Sleep one more sec to make sure the node announcement propagates.
 std::thread::sleep(std::time::Duration::from_secs(1));

 let expected_amount_sats = 100_000;
 let expiry_sec = 4_000;

 let uqr_payment = node_b.unified_qr_payment().receive(expected_amount_sats, "asdf", expiry_sec);
 let uri_str = uqr_payment.clone().unwrap();
 let offer_payment_id: PaymentId = match node_a.unified_qr_payment().send(&uri_str) {
  Ok(QrPaymentResult::Bolt12 { payment_id }) => {
   println!("\nBolt12 payment sent successfully with PaymentID: {:?}", payment_id);
   payment_id
  },
  Ok(QrPaymentResult::Bolt11 { payment_id: _ }) => {
   panic!("Expected Bolt12 payment but got Bolt11");
  },
  Ok(QrPaymentResult::Onchain { txid: _ }) => {
   panic!("Expected Bolt12 payment but get On-chain transaction");
  },
  Err(e) => {
   panic!("Expected Bolt12 payment but got error: {:?}", e);
  },
 };

 expect_payment_successful_event!(node_a, Some(offer_payment_id), None);

 // Removed one character from the offer to fall back on to invoice.
 // Still needs work
 let uri_str_with_invalid_offer = &uri_str[..uri_str.len() - 1];
 let invoice_payment_id: PaymentId =
  match node_a.unified_qr_payment().send(uri_str_with_invalid_offer) {
   Ok(QrPaymentResult::Bolt12 { payment_id: _ }) => {
    panic!("Expected Bolt11 payment but got Bolt12");
   },
   Ok(QrPaymentResult::Bolt11 { payment_id }) => {
    println!("\nBolt11 payment sent successfully with PaymentID: {:?}", payment_id);
    payment_id
   },
   Ok(QrPaymentResult::Onchain { txid: _ }) => {
    panic!("Expected Bolt11 payment but got on-chain transaction");
   },
   Err(e) => {
    panic!("Expected Bolt11 payment but got error: {:?}");
   },
  };
 expect_payment_successful_event!(node_a, Some(invoice_payment_id), None);

 let expect_onchain_amount_sats = 800_000;
 let onchain_uqr_payment =
  node_b.unified_qr_payment().receive(expect_onchain_amount_sats, "asdf", 4_000).unwrap();

 // Removed a character from the offer, so it would move on to the other parameters.
 let txid = match node_a
  .unified_qr_payment()
  .send(&onchain_uqr_payment.as_str()[..onchain_uqr_payment.len() - 1])
 {
  Ok(QrPaymentResult::Bolt12 { payment_id: _ }) => {
   panic!("Expected on-chain payment but got Bolt12")
  },
  Ok(QrPaymentResult::Bolt11 { payment_id: _ }) => {
   panic!("Expected on-chain payment but got Bolt11");
  },
  Ok(QrPaymentResult::Onchain { txid }) => {
   println!("\nOn-chain transaction successful with Txid: {}", txid);
   txid
  },
  Err(e) => {
   panic!("Expected on-chain payment but got error: {:?}");
  },
 };

 generate_blocks_and_wait(&bitcoind.client, &electrsd.client, 6);
 wait_for_tx(&electrsd.client, txid);

 node_a.sync_wallets().unwrap();
 node_b.sync_wallets().unwrap();

 assert_eq!(node_b.list_balances().total_onchain_balance_sats, 800_000);
 assert_eq!(node_b.list_balances().total_lightning_balance_sats, 200_000);
}
```

## SETTING UP THE TESTING ENVIRONMENT

We’ll start with the setup. We initialize the necessary components, including the bitcoin and electrum servers running on a regtest network, and set up two lightning nodes. This simulates the environment in which our payments will take place. Next, we need to generate and distribute bitcoin to a new address so we can fund our channel. Now, we open a payment channel between the two nodes and synchronize their wallets to reflect the updated state after the channel is opened. At this point, we now need to wait for the channel readiness events to ensure that the channel is properly established and both nodes are aware of each other.

## CREATE AND SEND A UNIFIED PAYMENT

Time to create a payment request and send some unified payments! First, we call the `unified_qr_payment().receive()` with Node B to generate the BIP21 unified payment URI. We have to clone the URI and call unwrap on it to get the URI as a string. With Node A we now can call `Unified_qr_payment().send(&uri_str)` which lets us send the payment given a URI! At this point, we verify that the BOLT12 payment is processed correctly and handle potential errors or incorrect payment types.

Next, we use the same process and URI string to test the fallback mechanism by attempting another payment. In this case, I deliberately remove one character from the URI, effectively invalidating the BOLT12 offer. While this test case isn’t ideal, given that BOLT12 is still in its infancy and has some unresolved bugs, it serves its purpose. If you’re interested in learning more about the bug, check out my last post linked in the first paragraph! Anyway, by modifying the offer, we simulate an invalid BOLT12 offer and verify that the fallback mechanism successfully defaults to a BOLT11 invoice. This confirms that the fallback functions as expected.

Finally, we finish the test by simulating a fallback to an on-chain payment when the offer is invalid and the channel liquidity is insufficient for the invoice. We’ll verify that the payment is correctly processed as an on-chain transaction by generating additional blocks to confirm it. Afterward, we check the node balances to ensure they reflect the expected amounts. This unit test evaluates the different payment scenarios and fallback mechanisms of the BIP21 Unified QR code handler, ensuring its robustness and reliability!

## UP NEXT

In my next post, I’ll dive deeper into the finer details of this project, sharing insights that didn’t make it into the other articles in the series. Beyond the code, I’ll also be discussing tips for building a standout resume as a community college graduate, and offering a guide to thriving while working from home — something that’s definitely been a learning curve for me! Anyway, thanks for joining me and if you found this article insightful or have any questions or critiques, I’d love to hear from you. Feel free to reach out to me on Twitter or LinkedIn. If you want to review my PR, check it out here.

```