---
layout: post
title: "So you want to persist your BDK wallet?"
date: 2025-08-12
author: "codingp110"  # Must match the name in _config.yml
categories: [BDK, Wallets, Development, Open-Source, Stories]  # See available categories below
image: ../assets/images/blog_content/2025-08-12-magical_bitcoin.png
---

### [`redb`](https://crates.io/crates/redb) for [`BDK`](https://github.com/bitcoindevkit)

---

This summer I have been working on implementing a persistence backend ([`bdk_redb`](https://crates.io/crates/bdk_redb)) for [`bdk_wallet`](https://crates.io/crates/bdk_wallet) and [`bdk_chain`](https://crates.io/crates/bdk_chain) structures using [`redb`](https://crates.io/crates/redb) as part of [`Summer Of Bitcoin'25`](https://www.summerofbitcoin.org/) . 
[`redb`](https://crates.io/crates/redb) is a pure Rust lightweight key-value database.

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-sob.png" alt="Summer Of Bitcoin logo" width="200" hspace="50" />
   <img src="../assets/images/blog_content/2025-08-12-bdk.png" alt="BDK logo" width="200" />
</p>

The motivation behind this is that the current persistence mechanism in [`BDK`](https://github.com/bitcoindevkit) is based on [`sqlite`](https://github.com/bitcoindevkit/bdk/blob/chain-0.23.0/crates/chain/src/rusqlite_impl.rs) which forces some application developers using [`redb`](https://crates.io/crates/redb) for their application data to use two types of databases. Also another goal is to replace the [`file_store`](https://crates.io/crates/bdk_file_store) which is the current persistence backend used by the team for testing.

### The Journey
In December, looking at [`BDK`](https://github.com/bitcoindevkit)'s friendly discord and proposal to incorporate [`silent-payments` in `BDK`](https://github.com/bitcoindevkit/bdk-sp) made me feel that I am at the right place! Fast forward to March 2025 when project proposals for [`BDK`](https://github.com/bitcoindevkit) came up I decided to go with the [`redb`](https://crates.io/crates/redb) one since both databases and networking were new to me (math major here!) and I had spent more time with the [`wallet`](https://github.com/bitcoindevkit/bdk_wallet/tree/release/2.0/wallet/src/wallet) code. I really did not realize the importance of the project at that time. Fast forward to May 2025 I started working on the project after the first call with my mentor, [`notmandatory`](https://github.com/notmandatory).

> 
>The main structure to be persisted is the [`Wallet`](https://docs.rs/bdk_wallet/2.0.0/bdk_wallet/struct.Wallet.html) which depends on structures from the [`bdk_chain`](https://crates.io/crates/bdk_chain): the [`TxGraph`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/tx_graph/struct.TxGraph.html), the [`Indexer`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/indexer/trait.Indexer.html) and the [`LocalChain`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/local_chain/struct.LocalChain.html). 
>[`TxGraph`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/tx_graph/struct.TxGraph.html) holds the transactions provided by the chain source like electrum, esplora and bitcoind. The [`Indexer`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/indexer/trait.Indexer.html) is used to index the transactions in the [`TxGraph`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/tx_graph/struct.TxGraph.html) and the [`LocalChain`](https://docs.rs/bdk_chain/0.23.0/bdk_chain/local_chain/struct.LocalChain.html) holds the block data provided by the chain source.
>Now, it is clearly wasteful to persist the whole structure each time it changes so rather we have the concept of `ChangeSet`. As the name suggests this structure records the changes to the corresponding structure. Then these `ChangeSet`s are persisted in memory. So functions in [`bdk_redb`](https://crates.io/crates/bdk_redb) take in `ChangeSet`s rather than the actual structures!
>Along with these we also need to persist the [`Network`](https://docs.rs/bitcoin/0.32.6/bitcoin/network/enum.Network.html) and the descriptors corresponding to the wallet

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-progress.png" alt="The image titled “PROGRESS” illustrates the difference between how we perceive progress and how it actually unfolds. On the left side, under the caption “HOW WE THINK IT IS EVOLVING,” a person is shown standing on a flat circular path, endlessly looping around the same level. This suggests that we often feel like we are going in circles without making any real headway. In contrast, the right side, labeled “HOW IT ACTUALLY IS EVOLVING,” depicts a person walking along an upward spiral. This symbolizes that despite the repetitive nature of our efforts, we are gradually moving forward and upward, growing and improving with each loop. The artwork, signed “@LINESBYLOES,” conveys an encouraging reminder that progress is not always linear or immediately visible, but it is happening nonetheless." width="400" />
</p>

Honestly, the project involved a lot of back and forth wherein we started with persisting the whole `ChangeSet`s as it is. After the first iteration, [`notmandatory`](https://github.com/notmandatory) suggested that we could do the persitence more smartly like in [`rusqlite`](https://github.com/bitcoindevkit/bdk/blob/chain-0.23.0/crates/chain/src/rusqlite_impl.rs) by persisting each individual fields of `ChangeSet`s instead since some fields may change more frequently than others. 

The next challenge was to decide the Rust types for the key and value in our [`redb`](https://crates.io/crates/redb) database.

> 
>Conceptually, a [`redb`](https://crates.io/crates/redb) database consists of tables which contain data in the form of key-value pairs. To use a certain structure as key and value we need to implement [`Key`](https://docs.rs/redb/2.6.0/redb/trait.Key.html) and [`Value`](https://docs.rs/redb/2.6.0/redb/trait.Value.html) trait respectively.

Initially we used primitive types (for which [`Key`](https://docs.rs/redb/2.6.0/redb/trait.Key.html) and [`Value`](https://docs.rs/redb/2.6.0/redb/trait.Value.html) trait are already implemented) to accomplish this and then after a suggestion from [`notmandatory`](https://github.com/notmandatory) we created wrappers around the structures we wanted to use as [`Key`](https://docs.rs/redb/2.6.0/redb/trait.Key.html) and [`Value`](https://docs.rs/redb/2.6.0/redb/trait.Value.html) and implemented the required traits for the same. Since the referential relationship between tables is unique to relational databases we tried replicating the behavior in [`bdk_redb`](https://crates.io/crates/bdk_redb) using external logic. 

Since we wanted to support multiple wallets in one db file we were forced to iterate over multiple irrelevant entries in a table which would clearly cause a performance issue. But my mentor in a masterstroke, suggested that we create a new table per structure and per wallet. We checked [`redb`](https://crates.io/crates/redb)'s issues page and found out that performance is log(no. of tables) so this should not be an issue. 

Owing to [`BDK`](https://github.com/bitcoindevkit)'s focus on modularity we tried making [`bdk_redb`](https://crates.io/crates/bdk_redb) as independent of [`bdk_wallet`](https://github.com/bitcoindevkit/bdk_wallet) as possible by using feature flags and using [`bdk_chain`](https://crates.io/crates/bdk_chain) structures wherever possible. This would allow users of [`bdk_chain`](https://crates.io/crates/bdk_chain) structures to also use [`bdk_redb`](https://crates.io/crates/bdk_redb) to persist their data.

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-frustracean.png" alt="This image is a comic with four panels. The first panel shows the Rust programming language logo and a happy crab labeled “Crustacean.” The second panel shows the crab using a laptop with the Rust logo, now called a “Rustacean.” The third panel shows the crab staring at a computer screen displaying the word “ERROR.” The final panel shows the crab looking angry with raised claws, labeled “Frustracean,” combining “frustrated” and “crustacean.” The comic humorously depicts how Rust programmers, nicknamed “Rustaceans,” often become frustrated when debugging errors." width="400" />
</p>

Then came the error handling to replace the clones and unwraps. It is now that I realized that implementing [`Key`](https://docs.rs/redb/2.6.0/redb/trait.Key.html) and [`Value`](https://docs.rs/redb/2.6.0/redb/trait.Value.html) for the wrapper types does not allow us to return serialization and deserialization errors! So back to primitive types...  
I initially started out with nested errors but while incorporating [`bdk_redb`](https://crates.io/crates/bdk_redb) into [`bdk-cli`](https://github.com/bitcoindevkit/bdk-cli/pull/206)  I decided it would be cleaner for downstream users if we [`remove the nesting`](https://github.com/110CodingP/bdk_redb/commit/b6f2ba4049c020fbfa3de87c8d7e80417bd4605e).
> 
> Panics are tools for developers and correspond to invariants which should never be broken by the code!

Afterwards due to my mentor's suggestions we decided to modify the API to use references to databases instead of a [`redb::Database`](https://docs.rs/redb/2.6.0/redb/struct.Database.html) since that helps many applications to use the same [`redb::Database`](https://docs.rs/redb/2.6.0/redb/struct.Database.html) which was one of the motivations! We used `Arc` since it is thread-safe.

From the initial stage itself [`notmandatory`](https://github.com/notmandatory) advised me to have tests so that we have something working end to end. And this helped a lot! I felt a lot more confident introducing changes into my persistence functions since I knew that I have tests written already. Code-coverage tools are a great way to aid testing as they identify branches (which turned out helpful for me), regions, lines which are not being tested by the tests!
> 
> Unit tests should be focused on the function/scenario they are trying to test. It is much cleaner and maintainable to create a common test setup for the unit tests.

---
### Learnings

At the start of the internship I was of the opinion that the production ready code that we see in [`BDK`](https://github.com/bitcoindevkit) or [`Bitcoin Core`](https://github.com/bitcoin/bitcoin) is what a BOSS dev would come up with as soon as they seen an issue but I was so wrong. I learnt that great code actually happens in iterations where the first few iterations are for the POC stage and the rest build up to production ready code!

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-rust.png" alt="Puzzled Ferris" width="200" />
</p>

One of the outcomes of the project for me is increased proficiency in Rust. Before the project, I only knew the basic syntax and how to use the [`docs`](https://docs.rs/) to find out relevant functions for the job! But the project made me go through multiple chapters from [`TRPL`](https://doc.rust-lang.org/book/)(the Holy Book of Rust) like Traits, Generics, Macros, Lifetimes, Concurrency, Error Handling, Closures etc., not to mention the countless blogs and [`Rust forum`](https://users.rust-lang.org/) questions! 
> 
> If you are ever stuck in implementing a trait take a look at how it has been implemented for the primitive types.

I learned seeing [`BDK`](https://github.com/bitcoindevkit)'s codebase that generic design has more to it than just Rust generics! Rather than a collection, the parameter to a function should be an iterator  so that any collection can be passed by converting to iter. A practice promoting forward compatibility is to use bounds when doing impl instead of applying bounds in the struct definition!

I also gained confidence in the [`BDK`](https://github.com/bitcoindevkit) codebase. Earlier I had no idea how different parts of [`BDK`](https://github.com/bitcoindevkit) interact and was particularly scared of [`bdk_chain`](https://crates.io/crates/bdk_chain) but now after spending some time persisting these parts :sweat_smile: and going over PRs on the repo I feel better about [`bdk_chain`](https://crates.io/crates/bdk_chain) :). The whole `ChangeSet` concept is actually very beautiful!

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-git.png" alt="This image is a black-and-white comic featuring three stick figure characters having a conversation. The first character, standing next to a desk with a laptop, says, “This is Git. It tracks collaborative work on projects through a beautiful distributed graph theory tree model.” The second character responds, “Cool. How do we use it?” The first character answers, “No idea. Just memorize these shell commands and type them to sync up. If you get errors, save your work elsewhere, delete the project, and download a fresh copy.” The comic humorously highlights how many developers feel about the confusing and sometimes arcane nature of using Git." width="400" />
</p>

Trying to maintain the [`repo`](https://github.com/110CodingP/bdk_redb) and keeping the commit history clean made me learn almost all common git commands: `pull`, `fetch`, `push`, `add`, `commit`, `remote`, `squash`, `rebase`, `cherry-pick` etc. 
> 
> If you want to identify whether a particular use case is supported by a library, try to look in the repo's open/closed issues.

Also being organized with questions and all the work done in the past week helps in dev-calls as it saves the time spent and back-and-forth done in trying to recall what all happened. 

---
#### Work Done So Far

Currently [`bdk_redb`](https://crates.io/crates/bdk_redb) contains a [`Store`](https://docs.rs/bdk_redb/0.1.0/bdk_redb/struct.Store.html) wrapper around a [`redb`](https://crates.io/crates/redb) database with methods to persist and read [`BDK`](https://github.com/bitcoindevkit) structures. [`Store`](https://docs.rs/bdk_redb/0.1.0/bdk_redb/struct.Store.html) also implements
[`WalletPersister`](https://docs.rs/bdk_wallet/2.0.0/bdk_wallet/trait.WalletPersister.html) under the wallet feature flag which allows [`bdk_wallet`](https://github.com/bitcoindevkit/bdk_wallet) users to use the [`Store`](https://docs.rs/bdk_redb/0.1.0/bdk_redb/struct.Store.html) for persistence. This [`Store`](https://docs.rs/bdk_redb/0.1.0/bdk_redb/struct.Store.html) can be used in place of the [`sqlite`](https://github.com/bitcoindevkit/bdk/blob/chain-0.23.0/crates/chain/src/rusqlite_impl.rs) backend currently supported by [`BDK`](https://github.com/bitcoindevkit/bdk). We support persisting multiple wallets in a single database file. The crate has been [`published on crates.io`](https://crates.io/crates/bdk_redb) now. Although the crate is still EXPERIMENTAL. DONOT use with MAINNET wallets. 
Also opened a PR to integrate [`bdk_redb`](https://crates.io/crates/bdk_redb) into [`BDK`](https://github.com/bitcoindevkit)'s playground called [`bdk-cli`](https://github.com/bitcoindevkit/bdk-cli/pull/206). 

---
### Looking Back

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-thankyou.png" alt="Minion saying Thank You" width="300" />
</p>

Honestly it was a marvelous feeling publishing my first crate (after starting out from scratch)! I hope this project will help [`BDK`](https://github.com/bitcoindevkit) users and the Bitcoin ecosystem :)

Also I am really thankful for such an [`awesome mentor`](https://github.com/notmandatory) ! Getting this far wouldn't have been possible without his constant help and guidance. 
Thank you [`Adi`](https://x.com/adi_shankara_) and [`SOB`](https://www.summerofbitcoin.org/) for such an awesome project and helping me get one step closer to my BOSS dream!

To all college students reading this, I highly recommend [`Summer Of Bitcoin`](https://www.summerofbitcoin.org/), a chance to work with awesome people on an awesome tech that is helping people around the globe!

---
### What Next ?
For developers implementing their own persistence crates it would be helpful if there is a generic persitence testing crate so that they do not need to write their own tests! The unit tests in [`bdk_redb`](https://crates.io/crates/bdk_redb) can be extracted out to make this testing crate! This is what I have been working on lately.

---

So bye! See you next time! Till then,
> Whoever you are whatever you do, do it for Bitcoin and Bitcoin will find you!

<p align="center">
   <img src="../assets/images/blog_content/2025-08-12-magical_bitcoin.png" alt="Magical Bitcoin logos" width="200" />
</p>