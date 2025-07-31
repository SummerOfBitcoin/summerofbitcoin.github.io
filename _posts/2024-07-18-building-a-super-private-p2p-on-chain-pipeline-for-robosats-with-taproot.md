---
layout: post
title: Building a Super Private P2P On-Chain Pipeline for RoboSats with Taproot
date: 2024-07-18
author: Aarav Mehta
categories: [Stories]
image: https://cdn.hashnode.com/res/hashnode/image/upload/v1721230211498/dfe93ace-5cf5-4572-b591-95f283e13b42.png
---

## Abstract

Hello there! I'm Aarav Mehta, studying Maths and Computing at IIT BHU. This year, I got the chance to participate in Summer of Bitcoin '24 in the Robosats organization, for a very exciting project of building Super private P2P onchain pipeline for RoboSats using Taproot Transaction, along with a very talented partner, f321x.

## My Project

RoboSats is a simple FOSS tool to privately exchange bitcoin for national currencies. Robosats simplifies the peer-to-peer user experience and uses lightning hold invoices to minimize custody and trust requirements. Anyone can be a robosats user and any lightning node can become a robosats provider, no barriers no costs.

My basic task was to use taproot contracts to build a super-private p2p onchain pipeline that will look just like a basic P2TR if the trade goes well (no disputes), using BDK library.

The task was very difficult, there are loads of different ways to build a taproot transaction, and there is very little material (compared to any other type of bitcoin transaction) online. Along with that, there was the matter of handling the data pipeline, and creating a proper user flow, so that the makers of requests and takers of requests of bonds can seamlessly interact with each other, and create their transactions.

Here is the link: https://github.com/RoboSats/taptrade-core

## Work Done

**Making the Architecture :** This was a very interesting challenge, as I didn't know much about how Taproot works before this. I had to read up so much, about so many different things in Bitcoin development, before I could make this. I will mention some of the main things I read up on before starting:

*   **Lightning Networks:** This is a very important concept, without which normal transactions could probably not exist in the real world, as the fees would be too high. I found this useful, since Robosats uses this in all its transactions, so it gave me the necessary background
*   **BIPs 340, 341, 342 (Taproot), BIP 327 (Musig), BIP 32 (xpub and xprv):** These are the Bitcoin Improvement Protocols which introduced the world to taproot and other stuff. This is basically the bible, which allow us to get into the minds of the brilliant developers who introduced us to these concepts and learn from them. While this has a lot of mathematics (which frankly went over my head), this did give me an introduction to what is taproot, why do we need it and how it can be used in a rough manner.
*   **Descriptors and Miniscript:** This is finally the essence of Taproot, as Descriptors allow us to specify how we want the contract to look like, and write the conditions about how bitcoin can be spent out from. This is written in a language called Miniscript. I have used this to create the taptree used in the final transaction
*   **Taproot contract implementations:** There are a lot of implementations for taproot contracts online. However, not all are relevant for the task at hand here. However we can realize that all taproot transactions have essentially the same common steps, which were finally included in the final architecture.
*   **HTLCs and PTLCs:** HTLCs or Hashed Timelock Contracts are used by Robosats currently. However all trades are not possible due to the short expiry times of routable HTLCs. (which is the motivation for this project). These are a very interesting concept, and I fell in love with the encryption algorithms which are used here. I believe the PTLC encryption algorithms using adaptor signatures can be used to design a more secure trade, and I have outlined a potential approach in the official obsidian file. However, currently, we are using the normal standard approach, and the other approach can be looked at later.
*   **Robosat issues:** There have been multiple issues and discussions related to this, not only in Robosats, but in bisq too. So the final architecture was made by the inspiration of these issues: bisq/5430, bisq-network/265, bisq-network/279, robosats/230, robosats/1114.
*   **Discreet Log Contracts:** Discrete Log Contracts are essentially used to handle betting, by taking in data from oracles to bet the final outcome. I think the negotiation procedure is extremely secure, and we can take future inspiration for updating the pre-Taproot negotiaition from the negotiation procedure used here.

**FINAL ARCHITECTURE**


**Making a HTTP Rest APIs Request, and achieving real time communication**

I had to make a web socket server, as a coordinator, so that I could respond to API requests made by the traders, and for that I had to choose between these 2 servers: Axum and Actix. Finally, I chose Axum, as the tokio people are maintaining it, so right now, that seemed like a solid choice. It is an exciting concept, and eventually efficient communication was able to be achieved.

This is how I achieved it:

'''rust
pub async fn api_server(coordinator: Arc<Coordinator>) -> Result<()> {
    let database = Arc::clone(&coordinator.coordinator_db);
    let wallet = Arc::clone(&coordinator.coordinator_wallet);
    let app = Router::new()
        .route("/create-offer", post(receive_order))
        .layer(Extension(database))
        .layer(Extension(wallet));
    let port: u16 = env::var("PORT")
        .unwrap_or_else(|_| "9999".to_string())
        .parse()?;
    info!("Listening on {}", port);
    let addr = SocketAddr::from(([127, 0, 0, 1], port));
    let tcp = TcpListener::bind(&addr).await.unwrap();
    axum::serve(tcp, app).await?;
    Ok(())
}
'''

**Using SQLite to handle orderbook and the different requests for the Bond Makers and Takers, and writing Unit Tests for testing this**

To ensure matching the maker and the taker, SQLite is used. This way we can store all the requests in SQLite tables. This was very fun for me to read about, since I had only studied the SQLite commands, how it is setup and its theory, but I was able to see it in action for the first time.

To ensure that everything is working correctly, I added a lot of tests, for all functions, by creating SQLite tables in :memory:, calling a lot of functions in it, and meticulously testing all functions, and using asserts to ensure that the correct outputs are being returned..

**Creating a Descriptor, with 6 different Alternate Spending Paths the Transaction can take to resolve the Taproot Transaction :**

I had to create a descriptor, and specify the six paths that can be used to withdraw funds from the transaction. For this, I needed to create a taptree, which can essentially be used to specify the alternate spending paths (other than the spending using the internal key that is).

I have done this essentially like this:

'''rust
let script_c: String = format!("and(pk({}),pk({}))", maker_key, coordinator_key);
let compiled_c = Concrete::<String>::from_str(&script_c)?.compile::<Tap>()?;
let tap_leaf_c = TapTree::Leaf(Arc::new(compiled_c));
let tap_tree = TapTree::Tree(Arc::new(tap_leaf_c), Arc::new(tap_leaf_d));
let internal_key = coordinator_key.to_string();
let descriptor = Descriptor::new_tr(internal_key, Some(tap_tree))?;
debug!("descriptor is {:}", descriptor);
'''

**Creating a bdk wallet and correspondingly a Partially Signed Bitcoin Transaction, that is essentiaily the final Taproot Transaction that is broadcasted as a P2TR Transaction**

For this part, I am firstly using Sparrow wallet and have created 3 wallets for testing: coordinator , taker and maker. I used BIP39 to derive my xpub and xprv keys, so that I can use them to conduct transactions and used testnet3 and 4 to get UTXOs from a faucet, and conducted sample transactions, so that these wallets can be used to fund and create different taproot psbt's.

Now, to create a PSBT from a descriptor, I created a new wallet with the descriptor described in the previous step and used the testnet wallet to fund this wallet. This way, the transaction is funded, and a new PSBT can be made, ready to be broadcast, and retrieve the funds on the basis of whichever condition is applicable

Here is how a transaction used to fund the wallet looks like: (in sparrow wallet):

Here is how the PSBT is finally made:

'''rust
let mut tx_builder= wallet.build_tx();
tx_builder
    .drain_wallet()
    .drain_to(address.script_pubkey())
    .fee_rate(FeeRate::from_sat_per_vb(3.0))
    .policy_path(BTreeMap::new(), KeychainKind::External);
let (psbt, tx_details) = tx_builder.finish()?;
debug!("PSBT: {:?}", psbt);
'''

**Achieved creation of the PSBT using Testnet3 and Testnet4 funds using the Sparrow Wallet, using different Backends like: Esplora Blockchain, Electrum Backend and Bitcoin node (setting up a server in the main repository itself)**

Currently, testnet3 has been reset, and will be completely depreciated in a few months and testnet4 is now currently active.

I have experimented by using different backends as Esplora doesn't support Testnet4, so I had to use Electrum for now instead. The plan is to use Bitcoin Node later, to have more control over the process.

## Challenges

Participating in the Summer of Bitcoin 2024 and working on the Robosats project has been an incredible learning experience, but it has not been without its challenges. The major challenge I had faced was **Understanding Taproot and Bitcoin Development**

Before starting this project, my knowledge of Taproot and Bitcoin development was limited. The complexity of Taproot transactions and the scarcity of comprehensive resources made the learning curve steep. I had to delve deep into Bitcoin Improvement Proposals (BIPs) and various technical documents to grasp the concepts.

## Post Mid-term

My plan for post Midterm is to:

*   Complete the taproot transaction, by adding the functionality for traders to give up their UTXO's
*   Making multiple structured Pull Requests, such that everything can be integrated and we are able to get feedback on the projects
*   Add integration testing to make sure that the trades are working correctly.
