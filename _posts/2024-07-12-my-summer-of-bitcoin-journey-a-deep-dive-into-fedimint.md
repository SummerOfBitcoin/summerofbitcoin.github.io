---
layout: post
title: "My Summer of Bitcoin Journey: A Deep Dive into Fedimint"
date: 2024-07-12
author: "Harsh Pratap Singh"
categories: [Stories, Fedimint]
---

Anyone who is into investing their hard earned money knows about Bitcoin, it's one thing that has gained significant traction over the years due to its phenomenal returns and dependability. I was too enthusiastic about it, but wanted to accumulate some technical knowledge before pouring in my money! I dont trust, I verify :) Then I stumbled upon Summer of Bitcoin which was a summer internship for university students around globe to learn about Bitcoin and earn a stipend for their OSS work after learning! Now thats what I can being at the right place at the right time! Less than 1% applications are accepted, its extremely competitive and the challenge that I had to solve was mind numbing to say the least, especially because I had no knowledge of Bitcoin technical details, so I learnt on the go (Programming Bitcoin and Grokking Bitcoin were a big help)! Luckily, I got selected to work on an Escrow Module based on paper for the Fedimint ecosystem, under the Summer of Bitcoin 2024, and I must say, I got really humbled and inspired looking at those Staff Software Engineers punching OSS code left, right and center! I mean the maintainers build at the speed of light :). I felt like this and I am grateful for the experience! I was aware of the some plugin/module architectures due to my work at Jenkins. The whole idea is to asyncronyously extend the backend without tinkering with the core, giving the devs the flexibility to empower the ecosystem! The Fedimint Primer is a great starting point for anyone to have an overview! This paper is also a great point to start understanding things conceptually!

### WHAT'S FEDIMINT?

Imagine you and a group of friends want to create a small, private digital cash system, a bit like a community bank or a shared digital piggy bank. You want it to be secure and trustworthy, even if not everyone in your group is available 24/7, or (in a more serious scenario) if one person tries to cheat.

How do you all agree on:

*   Who has how much money?
*   Which transactions are valid?
*   The order in which transactions happened?

This is where Federations and Consensus come in. They provide the framework for a group of participants to collectively manage a system and agree on its state.

Imagine the Fedimint federation is your local community bank. The Fedimint Client is like your personal banking app for that specific bank. It's the software you run on your phone or computer that lets you interact with the federation's services.

### WHAT'S FEDIMINT TECHNICALLY?

Fedimint represents a sophisticated approach to scaling Bitcoin and enhancing privacy through federated chaumian e-cash. Its modular architecture, robust consensus mechanism, and Lightning Network integration make it a powerful tool for building decentralized financial applications

A Fedimint Federation is a group of server operators, called Guardians - trusted members of the federation, who run the Fedimint software (fedimintd). A majority of these Guardians need to be honest and reliable. The fedimintd serves as the main consensus code for processing transactions and providing a REST API. It's the heart of the Fedimint server implementation.

Fedimint uses AlephBFT consensus algorithm to achieve agreement among federation members about the state of system like :

*   Which Transactions are valid
*   The order in which these transactions occurred
*   The current state of any Modules (like the digital cash module)

The result of this consensus process is a shared ledger.

```rust
// fedimint-core/src/epoch.rs
/// All the items that may be produced during a consensus epoch
#[derive(Debug, Clone, Eq, PartialEq, Hash, Encodable, Decodable)]
pub enum ConsensusItem {
    /// Threshold sign the epoch history for verification via the API
    Transaction(Transaction),
    /// Any data that modules require consensus on
    Module(ModuleConsensusItem),
    // ... other variants
}
```

A ConsensusItem can be a Transaction or data specific to a Module (like an e-cash minting operation).

How a transaction is agreed upon :
(Diagram showing user submitting a transaction proposal to guardians, who use AlephBFT to reach consensus and update the ledger.)

On the server-side (fedimintd), a component often referred to as the "Consensus Engine" manages this whole process. It takes items submitted for consensus, works with AlephBFT to order them, and then processes the agreed-upon items.

Fedimint Client is responsible for managing secrets, assets, building transactions, talking to federation, tracking operations, using modeules, etc.

Fedimint has an SDK in place in fedimint-client for providing high-level operations for interacting with the Fedimint federation using a Dynamic api client DynGlobalApi and client side data, initialized by ClientModuleInit which will be implemented by each module. It provides utilities for building and submitting transactions and uses a state machine (client state machines are for writing client side logic that the module might need so that they can be executed in idempotent steps that can get persisted in the database. This is so that everything work even if a client device is being constantly being shut-down abruptly etc.

When you send e-cash or perform other operations that change ownership of assets, the client builds a Transaction. This transaction will contain:

*   Inputs: What you're spending (e.g., some of your e-cash notes).
*   Outputs: What's being created (e.g., new e-cash notes for the recipient, and potentially change notes for yourself).
*   Signatures: Cryptographic proof that you authorized the spending of your inputs.

Typically inputs and output of a given transaction have client side state machine. Sometimes some other state machines are needed as well. Typically a transaction is submitted to federation, and client needs to do something afterward - e.g. wait for the transaction to get accepted or rejected and do some corresponding things afterwards. The states in the state machine can be thought of "snapshots" saved into the database. The SDK also handles logging and error handling.

```rust
pub struct Client {
    // Configuration specific to this federation
    config: tokio::sync::RwLock<ClientConfig>,
    // Your client's local database
    db: Database,
    // Your unique root secret for this federation
    root_secret: DerivableSecret,
    // API for talking to the federation guardians
    pub(crate) api: DynGlobalApi,
    // Registry of client modules (e-cash, LN, etc.)
    pub(crate) modules: ClientModuleRegistry,
    // Executor for running state machines
    executor: Executor,
    // Task group for managing background tasks
    task_group: TaskGroup,
    // ... other fields
}
```

The modules are the lifeline of fedimint architecture! The Mint module (implementing federated e-cash), Wallet module (handling on-chain deposits and withdrawing) and the Lightening module (integration with the Lightening Network allowing users to send and receive Lightning payments using their e-cash balance) serve as the core modules on which the Fedimint ecosystem functions. The developers can extend the fedimint-core by adding more modules, like I am doing right now :)

```rust
// Add modules you want to use (e.g., e-cash, Lightning)
// client_builder.with_module(MintClientInit); // For e-cash
// client_builder.with_module(LightningClientInit); // For Lightning

let client_handle = client_builder
    .join(my_root_secret, federation_config, None /* api_secret */)
    .await?;
```

This join process initializes your client for that specific federation.

The finalize_and_submit_transaction function is a core part of the Client. It takes a partially built transaction, works with modules to complete it (e.g., by adding e-cash inputs to pay for it and creating change outputs), and then sets up a state machine (TxSubmissionStatesSM) to manage the actual submission to the federation and track its acceptance.

The ClientHandle manages the lifecycle of the Client. When the last ClientHandle is dropped (goes out of scope), it triggers a shutdown process for the client, stopping its background tasks and releasing resources. This is important for clean program termination. Most of your interactions will be through this handle.

```rust
// fedimint-client/src/client/handle.rs
pub struct ClientHandle {
    inner: Option<Arc<Client>>,
}
```

How client interacts with Federation :
(Diagram showing the interaction flow between a User App, FedimintClient, E-cash Module, Federation API, and the Client's Local DB for a BTC deposit.)

Every Fedimint module has two parts that work together:

*   ClientModule: This defines how your client application interacts with the features of that specific module. It lives within your Fedimint Client.
*   ServerModule: This defines how the federation's guardians manage the state and rules for that module. It lives within the fedimintd server software run by the guardians.

The ClientModule for e-cash (often called MintClientModule) is responsible for things like:

*   Knowing how to construct a request to get new e-cash notes.
*   Storing your e-cash notes securely in the client's database.
*   Knowing how to select the e-cash notes you own when you want to spend them.
*   Creating the "input" part of a Transaction that proves you own the notes you're spending.
*   Understanding how to interpret responses from the federation related to e-cash.

The ServerModule for e-cash (often called MintServerModule) runs on each guardian's server. It's responsible for:

*   Defining the rules for valid e-cash notes.
*   Issuing the cryptographic signatures (blind signatures) that create new e-cash notes when funds are deposited.
*   Verifying the "input" part of a Transaction when someone tries to spend e-cash notes. This includes checking for valid signatures and preventing double-spends.
*   Keeping track of which notes have already been spent (managing the module's state).
*   Participating in the Federation & Consensus process to agree on the validity of e-cash transactions.

```rust
// fedimint-server-core/src/lib.rs (Simplified)
#[apply(async_trait_maybe_send!)]
pub trait ServerModule: Debug + Sized {
    // ... other types ...

    /// This module's contribution to the next consensus proposal.
    async fn consensus_proposal<'a>(
        &'a self,
        dbtx: &mut DatabaseTransaction<'_>,
    ) -> Vec</* ... ConsensusItem type ... */>;

    /// Process a consensus item agreed upon by the federation.
    async fn process_consensus_item<'a, 'b>(
        &'a self,
        dbtx: &mut DatabaseTransaction<'b>,
        consensus_item: /* ... ConsensusItem type ... */,
        peer_id: PeerId,
    ) -> anyhow::Result<()>;

    /// Verify and process a transaction input related to this module.
    async fn process_input<'a, 'b, 'c>(
        &'a self,
        dbtx: &mut DatabaseTransaction<'c>,
        input: &'b /* ... Input type ... */,
        in_point: InPoint,
    ) -> Result<InputMeta, /* ... InputError type ... */>;

    /// Verify and process a transaction output related to this module.
    async fn process_output<'a, 'b>(
        &'a self,
        dbtx: &mut DatabaseTransaction<'b>,
        output: &'a /* ... Output type ... */,
        out_point: OutPoint,
    ) -> Result<TransactionItemAmount, /* ... OutputError type ... */>;

    // ... other methods like verify_input, audit ...
}
```

The spend flow :
(Diagram illustrating the process of spending e-cash, involving the App, FedimintClient, Mint Client Module, Federation API, and Mint Server Module, including validation and double-spend checks.)

The module creation process is eased via the custom-module-example template. You can play with the module in a nerdy way using just mprocs inside the nix environment, just create some users and have fun with your module!

### DB

It uses RocksDB as the high-performance embedded key-value storage engine (can be easily integrated into the Fedimint application without requiring a separate database server) due to its high performance, especially for write-heavy workloads (which is crucial for a financial system like Fedimint) and fast reads, optimistic transactions and more! Fedimint has customized RocksDB options to suit its specific needs, such as setting write buffer size (performance optimization), use custom prefix-keys and recovery mode. This is sweet to go deeper into RocksDB!

### NIX

The first hurdle I faced before contributing was to get myself comfortable with the reproducible Nix environments (due to my low storage on 256GB macbook air, I had to nix-collect-garbage -d a lot, deeply painful, but Nix does make the developer experience outstanding). Nix is extensively used in the Fedimint project for managing the development environment (Flakebox) , building the project, and deploying Fedimint instances, cross-compilations in CI builds, and more ofcourse! Also, debugging rust in Nix was an exciting adventure! I wrote Starting to pick up Nix a bit as a collection of resources that I loved!

### MONITORING AND OBSERVIBILITY

Fedimint implements a robust metrics collection and exposure mechanism using Prometheus. Each module has its own set of metrics. The global collection of metrics happens in a static, lazy-initialized Registery using a custom macro system. It utilizes various Prometheus metric types, including Histogram, IntCounter, Gauge, and their vector counterparts. An new thing is the predefined buckets for amount-related histograms, chosen to cover a wide range of Bitcoin amounts, from fractions of a satoshi to 100 million satoshis (1 BTC). Fedimint exposes its metrics through an HTTP endpoint, implemented using the Axum. As soon as the the fedimintd process starts, the metrics server also starts. Metrics are updated when the transaction is successful maintaining consistency between DB and metrics. It supports OpenTelemetry integration with Jaeger for distributed tracing. These can easily be visualized in Grafana!

The load testing tool measures and compares performance metrics across different runs, tracking performance changes over time over different configurations.

### PERFORMANCE ANALYSIS

Chrome tracing is implemented for integration tests (devimint is used for local fedimint environments) in Fedimint, generating a JSON file named trace-{UNIX_TIME}.json in the current working directory which can be opened and visualized using Perfetto. This setup allows developers to perform detailed performance analysis of integration tests by visualizing the execution timeline, identifying bottlenecks, and understanding the flow of operations within the Fedimint system during test runs. Also, fedimint-load-test-tool is for performance testing for federation, scalability assessment, gateway integration testing.

### WASM TESTS

The Wasm tests are crucial for ensuring that Fedimint's client-side operations work correctly in a browser environment, which is essential for web-based wallets or applications using Fedimint. This is a really cool thing that I saw implemented in a Bitcoin project for the first time!

It also has fuzz testing but thats quite standard, so wont write about it.

Damn, the repository is sooo big, it will take me an eternity writing all about it, it would be best for you to check out the repository yourself, don't forget to star it!

### I GOT RUSTIER :)

Naturally, was curious on why folks are using Rust at the first place! especially in blockchain. Was pretty excited. Played with async Rust, and explored journey of Rust to async programming. I got intimidated by the codebase initially due to extremely complex and long Rust Types but this tutorial helped me a ton! I went deep into visualizingn memory layout of Rust Datatypes just for fun! Learnt about Static and Dynamic dispatcher! You know, its slightly harder to write some Data Structures in Rust! My mentor suggested me to look into Rust Design Axioms and in general better Rust patterns and techniques to improve maintainability and overall code quality for which I referred to this talk. The most dreaded thing that I had to face was the Rust's overconservative Borrow Checker. Explored rust compile times as well! Got exposed to conditional compilation between native and WASM targets and handling runtime differences. Hah! just got gently pushed to Advanced Rust!

Got initiated to think about structuring my code and thinking about the system design. Also got some advise on code reviews.

The best Rust tutorial that I could find was this! The more I use Rust, the more impressed I am!

Laughingly, during the internship, instead of doing Test Driven Development, I ended up doing Error Driven Development ;(

### THANK YOU!

The fedimint community was exceptionally warm and welcoming to the new contributors. Special thanks to the maintainers KodyLow, Manmeet, David, Justin and my mentor Shaurya for making the experience a memorable one! I will cherish it forever, they initiated me to Bitcoin!

### BITCOIN STUFF I AM EXCITED ABOUT IN THE NEAR FUTURE!

Really pumped up on how FROST signature schemes paired with Dynamic and Verifiable Secret Sharing will make the multi-sig wallet experience buttery smooth! The most awaited would be to have backup wallets! So Cool, it will redefine the Bitcoin experience for the users while also improving security :) Also excited to see the use of Conditional Payments using Discrete Log Contract suitable in cases where 2 mutually distrusting parties bet on the results! Also, BitVM is a heart-throb! And work on Bitcoin L2 is also cool, and I think StarkNet might very realistically scale Bitcoin with Ethereum!

My future generation will inherit a superior money, Bitcoin!

### WANNA CONTRIBUTE TO BITCOINS GLORY?

This is a solid starter advise for anyone new! Get accustomed to Bitcoin Development Philosophy
