---
layout: post
title: "A Beginner's Guide to Setting Up an Eclair Lightning Node"
date: 2024-07-25
author: "Mohit Kumar"
categories: [Tutorials, Lightning Network]
---

Hi everyone! Today, we are setting up an eclair node. If you're wondering, "What is eclair?", it is an implementation of the Lightning Network. It's coded in Scala, eclair can be your doorway into the world of the Lightning Network.

### Why should you consider running an eclair node?

Well, if transacting with lower fees sounds appealing or you're keen on exploring the underpinnings of Bitcoin's Lightning Network, or maybe, you're a developer interested in contributing to the codebase - running an eclair node is for you.

In this guide, we're using `Ubuntu 22.04`, `Bitcoin Core 23.2`.

Let's start our journey with the first step - setting up Bitcoin Core(if you already haven't).

**Step 1: Setting the stage with Bitcoin Core in regtest**

1. Download Bitcoin Core 23.2 from its [**official webpage**](https://bitcoincore.org/bin/bitcoin-core-23.2/) and extract the files.
    
2. Create a new directory named `.bitcoin` and inside it, a `bitcoin.conf` file with the following configuration:
    

```bash
server=1
txindex=1
regtest=1

rpcuser=user
rpcpassword=YourGeneratedPassword

zmqpubrawblock=tcp://127.0.0.1:29000
zmqpubhashblock=tcp://127.0.0.1:29000
zmqpubrawtx=tcp://127.0.0.1:29001

fallbackfee=0.0002
```

1. Time to secure your operation. Run the [`rpcauth.py`](http://rpcauth.py) file inside the `/extracted-folder/share/rpcauth` to generate a password.  
    You can use `python3 rpcauth.py "username"` for it  
    Replace `YourGeneratedPassword` in the `bitcoin.conf` file with this generated password.
    
2. Set the path to environment variable for `bitcoind` and `bitcoin-cli`. Your `.bashrc` would contain a line like `export PATH=$PATH:/home/claddy/bitcoin-23.2/bin`
    
3. Now, bring Bitcoin Core to life by running `bitcoind`.
    
4. Create a wallet with the command `bitcoin-cli createwallet YourWalletName`.
    

Once you're comfortable, experiment with a few Bitcoin commands! If you want to experiment with a full node, instead of regtest, check out this [guide](https://bitcoin.org/en/full-node#).

**Step 2: The lightning setup - eclair Node**

1. First, download the latest version of the eclair node from the [releases](https://github.com/ACINQ/eclair/releases).
    
2. Extract the downloaded folder and create a `.eclair` directory(if it isn't by default).
    
3. Inside the `.eclair` directory, create an `eclair.conf` file with this content:
    

```plaintext
csharpCopy codeeclair.chain = "regtest"
eclair.node-alias=alias
eclair.server.port=9735

eclair.trampoline-payments-enable=true

eclair.api.enabled=true
eclair.api.port=8080
eclair.api.password=password
eclair.bitcoind.rpcport=18443
eclair.bitcoind.rpcuser=user
eclair.bitcoind.rpcpassword=YourGeneratedPassword
eclair.bitcoind.zmqblock="tcp://127.0.0.1:29000"
eclair.bitcoind.zmqtx="tcp://127.0.0.1:29001"
```

1. Make sure to append the `rpcuser` and `rpcpassword` that you generated earlier using `rpcauth.py`.
    
2. Again, help your system find its way by setting the path variable for the `bin` folder of the eclair node.
    
3. Finally, run [`eclair-node.sh`](http://eclair-node.sh)`Declair.datadir=/home/your_username/.eclair/`. You'll be prompted for a password, enter the one you set in the `eclair.conf` file. By default the value of the password is `password`.
    

And voila! You're running your very own eclair node. Want to see what's happening? Check the `.log` file in your directory. Get to know your node by running `eclair-cli getinfo` in a new terminal.

The [eclair API documentation](https://acinq.github.io/eclair/) is your friend if you want to explore further. Want to run multiple eclair nodes? Here's a [**GitHub link**](https://github.com/t-bast/lightning-cfg) for you. And remember, [eclair](https://github.com/ACINQ/eclair) and [Bitcoin](https://github.com/bitcoin/bitcoin) documentation are always there to help you out.

This marks the end of our journey, and now, you've become a part of the Bitcoin Lightning Network. Any hurdles along the way? Reach me on [twitter](https://twitter.com/claddy_k). Keep exploring, keep learning!

