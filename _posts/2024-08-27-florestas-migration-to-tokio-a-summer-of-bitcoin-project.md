---
layout: post
title: "Floresta's Migration to Tokio: A Summer of Bitcoin Project"
date: 2024-08-27
author: "Shashank Trivedi"
categories: [Stories, Floresta]
---

## **Introduction**

In the heart of Summer of Bitcoin '24, Floresta, our lightweight Bitcoin full node implementation, is undergoing a significant transformation. This blog post details our exciting progress in migrating from the async-std runtime to Tokio, the leading async runtime in the Rust ecosystem. This shift promises enhanced performance, stability, and seamless integration with a wider array of Rust libraries and tools.

## **Glossary**

Before diving into the technical details, let's first understand the terms that will be used throughout this discussion:

* **Floresta:** A lightweight Bitcoin full node implementation designed for efficiency and extensibility.
    
* **Floresta-wire:** The networking layer of Floresta, responsible for peer-to-peer communication, message handling, and network protocols.
    
* **Floresta-electrum:** The Electrum server implementation within Floresta, providing a lightweight way for clients to interact with the Bitcoin network.
    
* **Florestad:** A full node implementation built on top of libfloresta, offering a watch-only wallet and an Electrum server.
    
* **libfloresta:** A collection of reusable components for building Bitcoin applications, serving as the foundation for Florestad.
    
* **Peer:** A node in the Bitcoin network that Floresta connects to for data exchange and communication.
    
* **Peer struct:** A data structure within the `peer` module that represents a peer and its associated state and communication channels.
    
* **Floresta-wire, Florestad, Floresta Electrum:** Various crates and sections of Floresta codebase.
    
* **Actor model:** A concurrency model where independent actors communicate through message passing, used in Floresta to manage TCP streams and other asynchronous operations.
    
* **TcpStream:** A TCP network socket used for communication between peers in Floresta.
    
* **Tokio:** An asynchronous runtime for Rust, providing the foundation for Floresta's asynchronous operations and concurrency.
    

## **Why Tokio?**

Async-std, while functional, is no longer actively maintained. Tokio, on the other hand, is the gold standard for asynchronous Rust development. Its robust feature set, active community, and proven track record make it the ideal choice to future-proof Floresta.

## **Channels and Actor Model of Tokio**

In Rust, Tokio leverages channels and the actor model as core components of its asynchronous runtime. Channels facilitate communication between different parts of an application, while the actor model provides a structured way to manage concurrency and state.

**Channels:** Tokio offers two primary types of channels:

* **Unbounded Channels:** These channels have unlimited capacity, allowing senders to transmit messages without blocking, regardless of whether a receiver is ready. This is ideal for scenarios where data flow can be bursty or unpredictable.
    
* **Bounded Channels:** These channels have a fixed capacity. Senders will block if the channel is full until space becomes available. This is useful for controlling resource usage and ensuring backpressure.
    

**Actor Model:** The actor model in Tokio involves:

* **Actors:** Independent units of computation that communicate exclusively through message passing.
    
* **Message Handling:** Actors define message handlers that process incoming messages sequentially. This ensures that an actor's internal state is only modified in a controlled and predictable manner.
    

**How They Work Together:**

1. **Message Sending:** When an actor wants to communicate with another actor, it sends a message over a channel. The message is serialized and placed in the channel's queue.
    
2. **Message Receiving:** The receiving actor listens on the channel for incoming messages. When a message arrives, it is deserialized and passed to the appropriate message handler.
    
3. **State Management:** Actors maintain their own internal state, which is updated in response to messages. This state is isolated from other actors, preventing race conditions and data corruption.
    

**Benefits:**

* **Concurrency:** Tokio's channels and actor model enable efficient concurrent programming, allowing multiple tasks to run simultaneously without interfering with each other.
    
* **Scalability:** The actor model promotes a modular and scalable design, making it easier to manage complex asynchronous applications.
    
* **Robustness:** By isolating state and using message passing, the actor model helps prevent common concurrency issues like data races and deadlocks.
    

**In the context of Floresta:**

The migration to Tokio involved replacing `async-std`'s unbounded channels with Tokio's `unbounded_sender` and receiver. Additionally, an actor model was implemented to manage TCP streams effectively, overcoming the limitation of `tokio::net::TcpStream` not being cloneable. This design pattern ensures that TCP operations are handled in a structured and efficient way within the Tokio runtime.

## Key Accomplishments

Various sections of Floresta needed to be updated with tokio for migration, lets go through them one by one:

1. **Floresta-wire Transformation:**
    
    * **Tokio Channels:** Replaced async-std's unbounded channels with Tokio's unbounded_sender and receiver for improved communication throughout the codebase.
        
    * **Actor Model for TCP Streams:**  
        Overcame the lack of Clone for `tokio::net::TcpStream` by implementing an actor model. This design provides a structured way to manage TCP operations efficiently within Tokio.
        
    * **Peer and Actor;** Deferred connection logic to actor from the peer, enabling multiple actor creations without changing the peer struct. Disassociated `TcpStream` into reader and writer, using Reader for the Actor to read from the stream and Writer directly in the `Peer` struct to write to the stream.
        
    * **Removal of socket_reader:**  
        Removed `socket_reader` as the reader was directly included in the actor, reducing the number of processes run.
        
    * **Refactor Socks Implementation:**  
        Adapted the Socks implementation to align with the new actor model and leverage tokio-streams, paving the way for streamlined peer creation in the node.
        
2. **Floresta crate changes:**
    
    * **Dependency Swap:** Replaced `async-std` dependencies with their Tokio equivalents (e.g., `RwLock`, etc).
        
    * **Main Runtime Refactor:** Transferred runtime to Tokio from Async Std.
        
3. **Floresta Electrum:**
    
    * **Dependency Swap and actor model:** Replaced `async-std` dependencies with their Tokio equivalents (e.g., `unbounded_channel` instead of `sender`). Implemented actor model spinning `unbounded_channel` similar to peer in floresta-wire.
        
    * **SenderMessage enum:** Created SenderMessage enum to handle messages to send appropriately, enabling future upgradation as well.
        
    * **Actor-Based Reading and Writing:** Implemented reading and writing inside the actor by disassociating TcpStream into reader and writer and using select between the read and write operations to choose among them simultaneously.
        
    * **Main Loop Refactor:** Modified the main_loop function to utilize Tokio’s unbounded channels for efficient message passing.
        
4. **Florestad Integration:**
    
    * **Tokio Runtime:** Implemented the Tokio runtime changing the `RwLock`, `block_on` implementation to use tokio specifications.
        
    * **Manual Runtime Build:** Addressed issue where `[Runtime:tokio]` caused a crash by manually building the Tokio runtime and implementing a graceful shutdown with a stop signal.
        
    * **Unified Integration:** Seamlessly integrated the Tokio-enhanced Floresta-wire and Floresta-electrum components into Florestad.
        

These comprehensive changes culminated in the complete migration of Floresta to the Tokio asynchronous runtime environment, enabling the stable operation of servers within this framework.  
Pull Request regarding all this work can be viewed here: [https://github.com/vinteumorg/Floresta/pull/172](https://github.com/vinteumorg/Floresta/pull/172)

## **What's Next?**

With the successful migration to Tokio, several exciting avenues for improvement have opened up for Floresta:

* **Tokio Tracing:** Tokio's built-in tracing capabilities offer a powerful mechanism for building diagnostic and debugging tools for asynchronous programs. We plan to leverage this feature to create a comprehensive console that provides deep insights into Floresta's internal operations, making it easier to identify and resolve performance bottlenecks and other issues.
    
* **SSL Integration:** Electrum supports SSL connections, and integrating this feature into Floresta would enable secure remote connections to the server. This enhancement would significantly improve the security and privacy of Floresta's Electrum server, making it more suitable for production environments.
    

## **Collaboration and Teamwork**

Throughout this migration process, collaboration and teamwork have been instrumental to my success. I've had the privilege of working closely with mentor Davidson Souza, whose expertise and guidance have been invaluable. Davidson's insights into Tokio's intricacies and best practices have not only helped me navigate challenges but also inspired me to explore innovative solutions. His suggestions for code improvements have significantly enhanced the efficiency and robustness of my implementation.

Most of the issues and design decisions stated above were pointed out by him. He helped with choosing the best possible way to migrate small sections of the highly integrated codebase of Floresta. He even helped out with coding parts where I got stuck a few times.

I extend my sincere gratitude to Davidson for his mentorship and support.

## **Conclusion**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724754147441/fb2b1cfb-86f6-47e6-a51a-ef4db12d5b0a.jpeg align="center")

We're thrilled to announce that the migration of Floresta to Tokio is now complete! This transition marks a significant milestone in Floresta's development, solidifying its position as a reliable, high-performance Bitcoin full node implementation. With Tokio as our foundation, we're confident that Floresta is well-equipped to meet the evolving demands of the Bitcoin network.

This migration has not only enhanced Floresta's performance and stability but has also opened doors to exciting new possibilities. We're eager to explore these opportunities and continue improving Floresta to better serve the needs of the Bitcoin community. Stay tuned for updates on upcoming features and enhancements as we continue to push the boundaries of Bitcoin innovation!
