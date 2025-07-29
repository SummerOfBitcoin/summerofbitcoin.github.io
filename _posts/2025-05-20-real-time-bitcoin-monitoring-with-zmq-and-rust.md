---
layout: post
title: "Real-Time Bitcoin Monitoring with ZMQ and Rust"
date: 2025-05-20 00:00:00
author: "Divyansh Seth"
categories: ['Stories', 'Demand']
---

## Introduction

Bitcoin Core has a powerful feature that many developers overlook: ZeroMQ (ZMQ) notifications. This feature lets you receive real-time updates about new blocks, transactions, and other network events directly from your Bitcoin node.

In this post, we'll build a Rust service that connects to Bitcoin Core's ZMQ interface to monitor blockchain activity in real-time. We'll focus on the `sequence` notification type, which provides ordered events for mempool additions, removals, and block connections/disconnections.

**TL;DR:** We'll create a Rust application that uses ZMQ sequence notifications to track Bitcoin mempool and block events in real-time, with proper error handling and event backlog management.

## What is ZeroMQ?

ZeroMQ is a lightweight wrapper around TCP connections, inter-process communication, and shared-memory, providing various message-oriented semantics such as publish/subscribe, request/reply, and push/pull.

Bitcoin Core can act as a trusted "border router" that implements the bitcoin wire protocol, makes consensus decisions, and maintains the local blockchain database. However, there exists only limited service to notify external software of events like new blocks or transactions.

The ZeroMQ facility implements a notification interface through specific notifiers. Bitcoin Core supports several notification types:

* **rawblock** - Full serialized block data
    
* **hashblock** - Block hash only (lighter weight)
    
* **rawtx** - Full serialized transaction data
    
* **hashtx** - Transaction hash only (lighter weight)
    
* **sequence** - Ordered events for mempool and block changes
    

This read-only facility requires only connecting a ZeroMQ subscriber port in receiving software. It's not authenticated and has no two-way protocol involvement, so subscribers should validate received data.

ZeroMQ sockets are self-connecting and self-healing - connections between endpoints automatically restore after outages, and either end can start/stop in any order.

## Problem or Motivation

Most Bitcoin applications need to know when new transactions or blocks appear. The traditional approach involves:

* Polling Bitcoin Core's RPC interface every few seconds
    
* Making multiple API calls to check for updates
    
* Dealing with delays and missed events
    
* Wasting resources on unnecessary requests
    

This polling approach is inefficient and can miss critical events. ZMQ solves this by providing real-time push notifications with proper sequencing to detect lost messages.

## Solution Overview

Our solution uses Bitcoin Core's `sequence` notification type, which provides ordered events for:

* **A** - Transaction added to mempool
    
* **R** - Transaction removed from mempool
    
* **C** - Block connected to chain
    
* **D** - Block disconnected from chain
    

The system includes:

1. **Bitcoin Core with ZMQ enabled** - Publishing sequence notifications
    
2. **Rust ZMQ subscriber** - Receiving and parsing messages
    
3. **RPC integration** - Fetching detailed data for events
    

**Note:** If you want to avoid RPC calls entirely, you can use other ZMQ notification types like `rawblock`, `rawtx`, `hashblock`, or `hashtx` which provide the data directly in the ZMQ message without needing additional RPC requests.

## Prerequisites

Before we start, you'll need to install ZeroMQ on your system. Download it from the [official ZeroMQ website](https://zeromq.org/download/).

You'll also need Bitcoin Core running in testnet or regtest mode for development.

## Deep Dive

### Setting up Bitcoin Core with ZMQ

Configure Bitcoin Core to publish sequence notifications. Add this to your `bitcoin.conf`:

```apache
# Enable ZMQ sequence notifications
zmqpubsequence=tcp://127.0.0.1:28332

# Optional: Enable other notification types
zmqpubrawblock=tcp://127.0.0.1:28333
zmqpubrawtx=tcp://127.0.0.1:28334
zmqpubhashblock=tcp://127.0.0.1:28335
zmqpubhashtx=tcp://127.0.0.1:28336
```

While Bitcoin Core supports multiple ZMQ notification types, we'll focus on the `sequence` type because it provides ordered events for state changes. The sequence notification provides ordered events with this format:

```apache
| topic    | 32-byte hash | event | sequence | msg sequence |
|----------|--------------|-------|----------|--------------|
| sequence | <block/tx>   | C/D/A/R | 8-byte  | 4-byte      |
```

Where All transaction and block hashes use reversed byte order, matching RPC and block explorer formats, the event types are:

* **C** - Block connected
    
* **D** - Block disconnected
    
* **A** - Transaction added to mempool
    
* **R** - Transaction removed from mempool
    

### Writing the Rust Service

Dependencies in `Cargo.toml`:

```ini
[dependencies]
zmq = "0.10"
bitcoin = "0.31"
bitcoincore-rpc = "0.18"
```

Core structure:

```rust
use zmq::Context;
use bitcoin::{Txid, BlockHash};

fn spawn_zmq_events(rpc: Client, event_broadcaster: EventBroadcaster, zmq_url: String) {
    std::thread::spawn(move || {
        let context = Context::new();

        loop {
            // Create subscriber with auto-reconnect
            let subscriber = context.socket(zmq::SUB).expect("Failed to create socket");
            subscriber.connect(&zmq_url).expect("Failed to connect");
            subscriber.set_reconnect_ivl(0).expect("Failed to set reconnect");
            subscriber.set_subscribe("sequence".as_bytes()).expect("Failed to subscribe");

            // Main event loop
            process_events(&subscriber, &rpc, &event_broadcaster);
        }
    });
}
```

### Processing ZMQ Messages

The message parsing logic handles the specific sequence format:

```rust
fn process_events(subscriber: &zmq::Socket, rpc: &Client, 
                  broadcaster: &EventBroadcaster) {
    loop {
        // Get new ZMQ message
        let topic = subscriber.recv_msg(0).expect("Failed to receive topic");
        if topic.as_str().unwrap_or("") == "sequence" {
            let raw = subscriber.recv_bytes(0).expect("Failed to receive data");
            
            if let Some(event) = parse_sequence_event(&raw, rpc) {
                if broadcaster.send(event.clone()).is_err() {
                    // Handle error
                }
            }
        }
    }
}
```

### Parsing Sequence Events

The sequence message format requires careful parsing:

```rust
fn parse_sequence_event(raw: &[u8], rpc: &Client) -> Option<SequenceEvent> {
    if raw.len() < 33 { return None; }

    // Extract hash (reversed byte order)
    let mut hash_bytes = raw[0..32].to_vec();
    hash_bytes.reverse();

    // Extract event type
    let event = raw[32] as char;
    
    // Extract sequence number (8 bytes, little endian)
    let sequence = if raw.len() >= 41 {
        let seq_bytes: [u8; 8] = raw[33..41].try_into().ok()?;
        u64::from_le_bytes(seq_bytes)
    } else { 0 };

    match event {
        'A' => handle_mempool_add(hash_bytes, sequence, rpc),
        'R' => handle_mempool_remove(hash_bytes, sequence),
        'C' => handle_block_connect(hash_bytes, rpc),
        'D' => handle_block_disconnect(hash_bytes, rpc),
        _ => None,
    }
}
```

### Event Handlers

Each event type requires specific handling:

```rust
fn handle_mempool_add(hash: Vec<u8>, sequence: u64, rpc: &Client) -> Option<SequenceEvent> {
    let txid = Txid::from_slice(&hash).ok()?;
    let entry = rpc.get_mempool_entry(&txid).ok()?;
    
    Some(SequenceEvent::MempoolAdd {
        sequence,
        transaction: MempoolTransaction::from((txid, entry)),
    })
}

fn handle_block_connect(hash: Vec<u8>, rpc: &Client) -> Option<SequenceEvent> {
    let block_hash = BlockHash::from_slice(&hash).ok()?;
    let block = rpc.get_block(&block_hash).ok()?;
    
    let txids = block.txdata.iter()
        .map(|tx| tx.compute_txid().to_string())
        .collect();
    
    Some(SequenceEvent::BlockConnect {
        block: BlockTransactionList {
            block_hash: block_hash.to_string(),
            txids,
        },
    })
}
```

## Challenges and Learnings

Building this system taught me several important lessons:

**ZMQ Message Format Complexity**: The sequence notification format is more complex than other ZMQ messages. The hash bytes need to be reversed, and the sequence number parsing requires careful handling of little-endian byte order. I initially missed that mempool events include an 8-byte sequence number after the event type.

**Auto-Reconnection Logic**: ZMQ connections can drop, especially during Bitcoin Core restarts. The code implements automatic reconnection by recreating the socket in the outer loop. Setting `reconnect_ivl(0)` ensures immediate reconnection attempts.

**Event Backlog Management**: During network congestion or processing delays, events can accumulate faster than they're processed. The backlog queue prevents event loss by storing unprocessed events and prioritizing them on the next iteration.

**RPC Integration Timing**: Sometimes ZMQ notifications arrive before the RPC interface has the corresponding data available. This created race conditions where `get_mempool_entry()` calls would fail. Adding proper error handling and retry logic was essential.

**Byte Order Confusion**: Bitcoin uses reversed byte order for hashes in its RPC interface and ZMQ messages. The sequence events provide hashes in the same format, but you need to reverse them when creating `Txid` and `BlockHash` objects.

## Conclusion

We've built a robust real-time Bitcoin monitoring service using ZMQ sequence notifications and Rust. This system provides ordered, reliable access to mempool and block events without the inefficiency of polling.

The key advantages of this approach:

* **Real-time notifications** with proper event ordering
    
* **Automatic reconnection** handling network disruptions
    
* **RPC integration** for detailed event data
    

The sequence notification type is particularly powerful because it provides a complete view of blockchain state changes with guaranteed ordering, making it ideal for applications that need to maintain accurate state.

**Next improvements could include:**

* Implementing sequence number gap detection for lost messages
    
* Creating filtered subscriptions for specific transaction types or addresses
    
* Using other ZMQ notification types like `rawblock` or `rawtx` to eliminate RPC dependency entirely
    

This foundation provides everything needed for building responsive Bitcoin applications that react immediately to network events, whether you're building a wallet, exchange, or analytics platform.

## Resources and Further Reading

For more detailed information about Bitcoin Core's ZMQ implementation, message formats, and configuration options, check out the official documentation:

**Bitcoin Core ZMQ Documentation:** [https://github.com/bitcoin/bitcoin/blob/master/doc/zmq.md](https://github.com/bitcoin/bitcoin/blob/master/doc/zmq.md)

Remember to always test thoroughly with testnet before deploying to mainnet, and consider the security implications of exposing ZMQ endpoints beyond localhost.