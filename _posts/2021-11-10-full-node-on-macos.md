---
layout: post
title: Playing with Bitcoin CLI and Running a Full Node on MacOS
author: Sandipan Dey
date: "2021-11-10 13:27:00 +0000"
categories:
  - "Tutorials"
description: "A beginner's guide."
---

## Installation

Before we proceed, we are going to require [Homebrew for MacOS](https://brew.sh/?ref=blog.summerofbitcoin.org) to install many dependencies that are present in Linux Machines out of the box.

```shell
brew install wget
brew install gnupg
```

One should download and install the Bitcoin Core software directly from [https://bitcoincore.org/download](https://bitcoincore.org/download?ref=blog.summerofbitcoin.org). In this tutorial, we are going to download the `tar.gz` file of Bitcoin Core. The given script takes care of finding the latest version and downloads the same. Do start this script from the location you want to keep all the bitcoin related files at.

```shell
mkdir bitcoin && cd bitcoin

# Fetch the latest version
VERSION=$(curl -v --silent https://bitcoincore.org/en/download/ 2>&1 | grep -oh "[Ll]atest version: [0-9]*\.[0-9]*\.[0-9]*" | cut -d " " -f3)

# Download the tarball for MacOS
wget https://bitcoincore.org/bin/bitcoin-core-$VERSION/bitcoin-$VERSION-osx64.tar.gz

# Fetch the signature file
wget https://bitcoincore.org/bin/bitcoin-core-$VERSION/SHA256SUMS.asc

# Match signature
shasum -a 256 --check SHA256SUMS.asc

# Obtain a copy of the release signing key - signing key taken from https://bitcoincore.org/en/download
gpg --keyserver hkp://keyserver.ubuntu.com --recv-keys 01EA5486DE18A882D4C2684590C8019E36C2E964

# Verify PGP signature
gpg --verify SHA256SUMS.asc

# Unpack Bitcoin Core tarball
tar -zxvf bitcoin-$VERSION-osx64.tar.gz

# Deleting downloaded files
rm bitcoin-$VERSION-osx64.tar.gz SHA256SUMS.asc

# Adding binaries to PATH
cd bitcoin-$VERSION/bin
echo "export PATH=\$PATH:"$(pwd) >> ~/.zshrc
```

You should now find `bitcoind` and `bitcoin-cli` on the terminal.

### Configuration

On MacOS, the default configuration file for Bitcoin Core `bitcoin.conf` resides in the location `~/Library/Application\ Support/Bitcoin/bitcoin.conf`.

We are going to test things out on the Signet chain. Here is the configuration to do the same. Also, in case you have an external disk to store the blocks, you can add in a dedicated data directory for the same.

```shell
chain=signet
txindex=1
prune=550
datadir=/Volumes/External-SSD/bitcoin/.bitcoin
```

### Starting Bitcoin Core

```shell
# Start Bitcoin Core as a daemon process
bitcoind -daemon

# Watch blocks and data from the blockchain being downloaded
watch bitcoin-cli getblockchaininfo

# Stop Bitcoin Core
bitcoin-cli stop
```

### Create, Encrypt and Backup your own Wallet

Where's the fun of running a full node without a wallet? 😆

Bitcoin Core allows you to create a HD wallet but it doesn't give you the mnemonic phrase. Instead, you need to backup your wallet as a file.

```shell
# Creating the wallet
# Fetches the password from standard input, otherwise, bash history would have
# the password of the wallet in it. Enter a password and then hit ^D.
bitcoin-cli -stdin createwallet mywallet false false
[passphrase]
^D

# Fetching more details of the wallet
bitcoin-cli getwalletinfo

# Backing up your wallet
bitcoin-cli backupwallet ~/walletbackup.dat

# Unlock wallet for few seconds
bitcoin-cli -stdin walletpassphrase
[passphrase]
[no of seconds to keep it unlocked]
^D
```

### Receive & Send Transactions

```shell
# Always use fresh addresses for privacy, to get it from your HD wallet use
bitcoin-cli getnewaddress

# Fetch confirmed balance
bitcoin-cli getbalance

# Fetch all balances
bitcoin-cli getbalances

# List all transactions that deals with one or more of your addresses
bitcoin-cli listtransactions
```

You can use [Signet Faucet](https://signet.bc-2.jp/?ref=blog.summerofbitcoin.org) to send yourself some test coins. Paste the output of `getnewaddress` in this website and get 0.1 BTC.

```shell
bitcoin-cli getnewaddress
tb1q3sup5jxszdgazdknh4npla7s7hqm7ph46chl4q

bitcoin-cli listtransactions
[
  {
    "address": "tb1q3sup5jxszdgazdknh4npla7s7hqm7ph46chl4q",
    "category": "receive",
    "amount": 0.10000000,
    "label": "",
    "vout": 1,
    "confirmations": 0,
    "trusted": false,
    "txid": "8d8d32cdd3b3214b311d7685e94ca8f8404127f9e517c59652df5cc86439debe",
    "walletconflicts": [
    ],
    "time": 1625979269,
    "timereceived": 1625979269,
    "bip125-replaceable": "no"
  }
]
```

## Practice Exercises

Below are the exercises taken from [Kalle Rosenbaum's classes at Summer of Bitcoin '21](https://rosenbaum.se/book/cc/instructions.txt?ref=blog.summerofbitcoin.org) and the solutions to the same are discussed below in detail. Before proceeding, please check that you have atleast 1 BTC on Signet to begin with, lest you may try requesting them from Signet Faucet. Also, BTC is a valuable asset so remember the price implications when playing with Bitcoin.

### 1. Make Payments to Various Addresses

In this exercise, we are going to create two addresses and send 0.1 BTC from on address to another. Since we are rotating coins within our wallet, resultant sum of money must not go down significantly unless there is any transaction fees.

```shell
bitcoin-cli getbalance
5.10000000

bitcoin-cli getnewaddress
tb1qttars2jj673ze2hax92qe36y2pvkg3yu67lcz0

bitcoin-cli getnewaddress
tb1qf6ejual6fxy8e3c37jkmy3xp0xhwnzex63ppn2

bitcoin-cli getnewaddress
tb1qfszkxrrq95r7wyudxlhhsyu943q0ufreqk3l5m

# Before sending BTC, you need to unlock your wallet with your passphrase
bitcoin-cli -stdin walletpassphrase
[password]
[number of seconds to unlock wallet]
^D

# Sending 0.01 BTC to first address.
bitcoin-cli -named sendtoaddress address="tb1qttars2jj673ze2hax92qe36y2pvkg3yu67lcz0" amount=0.01
1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf

# You may have noticed the -named flag, this allows to pass name=value
# style arguments one can find on bitcoin-cli help.

# The above command returned the txid of the 0.01 BTC payment, so we can
# look into the tx in more details
bitcoin-cli gettransaction "1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf"
{
  "amount": 0.00000000,
  "fee": -0.00000182,
  "confirmations": 0,
  "trusted": true,
  "txid": "1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf",
  "walletconflicts": [
  ],
  "time": 1625982080,
  "timereceived": 1625982080,
  "bip125-replaceable": "no",
  "details": [
    {
      "address": "tb1qttars2jj673ze2hax92qe36y2pvkg3yu67lcz0",
      "category": "send",
      "amount": -0.01000000,
      "label": "",
      "vout": 0,
      "fee": -0.00000182,
      "abandoned": false
    },
    {
      "address": "tb1qttars2jj673ze2hax92qe36y2pvkg3yu67lcz0",
      "category": "receive",
      "amount": 0.01000000,
      "label": "",
      "vout": 0
    }
  ],
  "hex": "02000000000101bede3964c85cdf5296c517e5f9274140f8a84ce985761d314b21b3d3cd328d8d0100000000feffffff0240420f00000000001600145afa382a52d7a22caafd31540cc744505964449c8a538900000000001600140f8e319bfe65a51e8ba16f1e511b4b3e7158ead302473044022030459affdea0f5e80d08f751ab661b632377caf08c739e1c2d0f2321ca23939202206098b0089ac843d41afe5189b69df2c3c374625f3b7987cdb82a267a9aadf9c00121037949e3417da991f55c00b63aa807167da9e9d718546bc356de245ae5d2927e6fe9b30000"
}

# The amount here is 0.00000000 because there is not net difference is wallet
# balance after this transaction gets approved. Miners would get a tx fee of 
# 0.00000182 BTC which was auto calculated by Bitcoin Core. In details, it
# specifies that the address which sent and address which received. Normally,
# if others were to send this tx to you, you would only see the receive part
# but since in our example, because we have done both sending and receiving
# from our wallet, details contains both of them. This tx was not bip125
# replaceable which means we won't be able to bump tx fee later on.

# It's important to lock your wallet back
bitcoin-cli walletlock
```

### 2. Inspect a Transaction in Detail

We are going to inspect the last transaction we made in more detail. Here is the same transaction on [Signet Block Explorer](https://explorer.bc-2.jp/tx/1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf?ref=blog.summerofbitcoin.org).

```shell
bitcoin-cli getrawtransaction "1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf" true     
{
  "txid": "1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf",
  "hash": "33811d32129ab61c61623afec8164ba48f243973073c2e191c0b66f6fcdccaac",
  "version": 2,
  "size": 222,
  "vsize": 141,
  "weight": 561,
  "locktime": 46057,
  "vin": [
    {
      "txid": "8d8d32cdd3b3214b311d7685e94ca8f8404127f9e517c59652df5cc86439debe",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "3044022030459affdea0f5e80d08f751ab661b632377caf08c739e1c2d0f2321ca23939202206098b0089ac843d41afe5189b69df2c3c374625f3b7987cdb82a267a9aadf9c001",
        "037949e3417da991f55c00b63aa807167da9e9d718546bc356de245ae5d2927e6f"
      ],
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.01000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 5afa382a52d7a22caafd31540cc744505964449c",
        "hex": "00145afa382a52d7a22caafd31540cc744505964449c",
        "reqSigs": 1,
        "type": "witness_v0_keyhash",
        "addresses": [
          "tb1qttars2jj673ze2hax92qe36y2pvkg3yu67lcz0"
        ]
      }
    },
    {
      "value": 0.08999818,
      "n": 1,
      "scriptPubKey": {
        "asm": "0 0f8e319bfe65a51e8ba16f1e511b4b3e7158ead3",
        "hex": "00140f8e319bfe65a51e8ba16f1e511b4b3e7158ead3",
        "reqSigs": 1,
        "type": "witness_v0_keyhash",
        "addresses": [
          "tb1qp78rrxl7vkj3azapdu09zx6t8ec436knkjg7xd"
        ]
      }
    }
  ],
  "hex": "02000000000101bede3964c85cdf5296c517e5f9274140f8a84ce985761d314b21b3d3cd328d8d0100000000feffffff0240420f00000000001600145afa382a52d7a22caafd31540cc744505964449c8a538900000000001600140f8e319bfe65a51e8ba16f1e511b4b3e7158ead302473044022030459affdea0f5e80d08f751ab661b632377caf08c739e1c2d0f2321ca23939202206098b0089ac843d41afe5189b69df2c3c374625f3b7987cdb82a267a9aadf9c00121037949e3417da991f55c00b63aa807167da9e9d718546bc356de245ae5d2927e6fe9b30000",
  "blockhash": "00000144fcc98a47f87e52519d2d925e46538d2f97cfc12dcb2bf9822e57de31",
  "confirmations": 2,
  "time": 1625983108,
  "blocktime": 1625983108
}

```
Let's start with the inputs/outputs. The `vin` specifies the UTXO of `8d8d32`'s 1st Output. Since we are spending this, we need to sign this transaction which is done using Segwit - it is present in `txinwitness`. Both the outputs show that they were `p2wpkh` outputs and they indeed have the pubkey script with starting `00` as the version byte, followed by 20 byte witness program. Bitcoin Core must have created a new address for the change because the last `...7xd` address wasn't an address we created. The locktime was set to the most recent block height so that it gets mined now. The output also includes the `blockhash` of the block this tx got itself into. There is one more block after this one which is why there are 2 confirmations.

### 3. Get Merkle Proof of a Transaction and Verify

For this exercise, we are going to use [gettxoutproof](https://chainquery.com/bitcoin-cli/gettxoutproof?ref=blog.summerofbitcoin.org) and [verifytxoutproof](https://chainquery.com/bitcoin-cli/verifytxoutproof?ref=blog.summerofbitcoin.org). The output from `gettxoutproof` is a hex-encoded proof which has the capability to prove that our transaction is in a block. `verifytxoutproof` reads this hex-encoded proof and confirms whether or not our transaction was included.

```shell
# The input txid and output txid must match
bitcoin-cli gettxoutproof '["1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf"]' | bitcoin-cli -stdin verifytxoutproof
[
  "1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf"
]

# Just creating a transaction and checking verifytxoutproof output
bitcoin-cli -named sendtoaddress address="tb1qttars2jj673ze2hax92qe36y2pvkg3yu67lcz0" amount=0.01                                     
dbc239cd636a8603c04530fd8c0349acd3b935d74bc2bdc0f8ef5ade75df9e95

# It should say this tx is not yet in block
bitcoin-cli gettxoutproof '["dbc239cd636a8603c04530fd8c0349acd3b935d74bc2bdc0f8ef5ade75df9e95"]' | bitcoin-cli -stdin verifytxoutproof
error code: -5
error message:
Transaction not yet in block

# Trying a little later
bitcoin-cli gettxoutproof '["dbc239cd636a8603c04530fd8c0349acd3b935d74bc2bdc0f8ef5ade75df9e95"]' | bitcoin-cli -stdin verifytxoutproof
[
  "dbc239cd636a8603c04530fd8c0349acd3b935d74bc2bdc0f8ef5ade75df9e95"
]
```

### 4. Fee Bumping

On mainnet it's highly possible that a tx with a low amount of tx fees (~1sat/vB) would take a lot of time to get verified. Hence, BIP 125 introduced a feature of bumping transaction fees by adjusting their previously sent transaction fees. A transaction can opt-in to [Replace-By-Fee(RBF)](https://rosenbaum.se/book/grokking-bitcoin-9.html?ref=blog.summerofbitcoin.org#_opt_in_replace_by_fee). The sequence number should be less than `fffffffe` for a transaction to signal as opted into RBF. The user's wallet creates a new transaction spending the same UTXO as the old tx, but this time, the outputs are adjusted to account for a higher tx fees.

```shell
# Sending a transaction with low tx fees
bitcoin-cli -named sendtoaddress address="tb1qf6ejual6fxy8e3c37jkmy3xp0xhwnzex63ppn2" amount=0.5 fee_rate=1 replaceable=true
19925b51c85f1f569da065894b548606933195ef13aa96cb5fb258e5003756ec

# Bumping the fees of this transaction - the new fee gets automatically 
# calculated by Bitcoin Core using estimatesmartfee
bitcoin-cli bumpfee "19925b51c85f1f569da065894b548606933195ef13aa96cb5fb258e5003756ec"
{
  "txid": "90f6953a9057c8d8360c07b9fdd84613848acea6ece41bb2743fd8db57103c54",
  "origfee": 0.00000141,
  "fee": 0.00000846,
  "errors": [
  ]
}

# After some time, let's see the older tx with low fees
bitcoin-cli gettransaction "19925b51c85f1f569da065894b548606933195ef13aa96cb5fb258e5003756ec"
{
  "amount": 0.00000000,
  "fee": -0.00000141,
  "confirmations": -1,
  "trusted": false,
  "txid": "19925b51c85f1f569da065894b548606933195ef13aa96cb5fb258e5003756ec",
  "walletconflicts": [
    "90f6953a9057c8d8360c07b9fdd84613848acea6ece41bb2743fd8db57103c54"
  ],
  "time": 1625989689,
  "timereceived": 1625989689,
  "bip125-replaceable": "yes",
  "replaced_by_txid": "90f6953a9057c8d8360c07b9fdd84613848acea6ece41bb2743fd8db57103c54",
  "details": [
    {
      "address": "tb1qf6ejual6fxy8e3c37jkmy3xp0xhwnzex63ppn2",
      "category": "send",
      "amount": -0.50000000,
      "label": "",
      "vout": 0,
      "fee": -0.00000141,
      "abandoned": false
    }
  ],
  "hex": "02000000000101c3a48e33abf6c970e0a6852936db85bd192f98038065dc5f68193c4ba031bbb30000000000fdffffff0280f0fa02000000001600144eb32e77fa49887cc711f4adb244c179aee98b26f373d21a000000001600148312fd485c437a5b51c34e8186e07e274bcab3ce0247304402203636d4a68bbf9baf1311b8a6fe4c1330cfcbd7eb20e1a8f880b0949343f51213022075497c29d5ea335938934bbfd72f807ab15bd0b14cf998d91f21978ecdf339cb0121034ce89d0c57ad806866979439cee18563ef864f283ea40bbcc7608247f0094782c5b30000"
}

# As is clearly evident, there are negative confirmations which is a proof
# that this transaction is not going to be part of the chain at all. Also,
# note that this tx was bip125-replaceable.

# Let's check the newer transaction
bitcoin-cli gettransaction "90f6953a9057c8d8360c07b9fdd84613848acea6ece41bb2743fd8db57103c54"
{
  "amount": 0.00000000,
  "fee": -0.00000846,
  "confirmations": 1,
  "blockhash": "000000cc39cf16910d7fab3948e2aa2750d39572f8951d1a37c360ede01f506f",
  "blockheight": 46068,
  "blockindex": 1,
  "blocktime": 1625989976,
  "txid": "90f6953a9057c8d8360c07b9fdd84613848acea6ece41bb2743fd8db57103c54",
  "walletconflicts": [
    "19925b51c85f1f569da065894b548606933195ef13aa96cb5fb258e5003756ec"
  ],
  "time": 1625989698,
  "timereceived": 1625989698,
  "bip125-replaceable": "no",
  "replaces_txid": "19925b51c85f1f569da065894b548606933195ef13aa96cb5fb258e5003756ec",
  "details": [
    {
      "address": "tb1qf6ejual6fxy8e3c37jkmy3xp0xhwnzex63ppn2",
      "category": "send",
      "amount": -0.50000000,
      "label": "",
      "vout": 1,
      "fee": -0.00000846,
      "abandoned": false
    },
    {
      "address": "tb1qf6ejual6fxy8e3c37jkmy3xp0xhwnzex63ppn2",
      "category": "receive",
      "amount": 0.50000000,
      "label": "",
      "vout": 1
    }
  ],
  "hex": "02000000000101c3a48e33abf6c970e0a6852936db85bd192f98038065dc5f68193c4ba031bbb30000000000fdffffff023271d21a000000001600148312fd485c437a5b51c34e8186e07e274bcab3ce80f0fa02000000001600144eb32e77fa49887cc711f4adb244c179aee98b26024730440220260284c67940c8c0fd13b15f0fe128a564b0eebe3bb3afb1f4e7076bc4f898c3022044df3cc0a9aa9e7eeb16db6523d87c3e74103f9a1192ecee891ca2dab6aa3ecd0121034ce89d0c57ad806866979439cee18563ef864f283ea40bbcc7608247f0094782f3b30000"
}
```

### 5. Calculate Target from Block Header

In this exercise, we are going to calculate the target from raw block header. To do so, we figure out the target bytes, convert them to Big Endian order and decode the same as the target threshold. To convert the same to recognizable number we are going to follow the steps as in this [StackExchange answer](https://bitcoin.stackexchange.com/a/36228?ref=blog.summerofbitcoin.org)

```shell
bitcoin-cli getblockheader $(bitcoin-cli getbestblockhash) false
00000020b5355338355fa2f0f602418b3130e5b1024a673c4e22e22c9f231c763c010000e1c4a8bad633d75cf9345ac09f75d78fde8e65e2b2b2cb46faddf0817e89117af1a8ea600e5b011eb3908401
```

From [Grokking Bitcoin - Chapter 11.3.2](https://rosenbaum.se/book/grokking-bitcoin-11.html?ref=blog.summerofbitcoin.org#_using_incremented_block_version_number_signalingbip34_66_and_65), we know that target is the second last four bytes. Hence, the target bits will be:

```shell
0x0e5b011e
```

In Computers, data is stored in Little Endian notation, converting the same to Big Endian:

```shell
0x1e015b0e
```

Using the given StackExchange answer, we can calculate the mantissa and exponent.

```shell
0x015b0e * 2 ^ 8 ^ (0x1e - 3)
= 0x015b0e * 2 ^ (8 * 0x1b)
= 0x015b0e * 2 ^ (8 * 27) 
= 0x015b0e * 2 ^ 216
= 0x015b0e<216 0 bits>
= <16 0 bits>0x015b0e<216 0 bits> [Ans]
```

### 6. OP\_RETURN & Proof-of-Burn

If you've come this far, you would want to add a memorandum like "User was here" in the blockchain because it's going to stay there forever.

For this exercise, we would need to work with the raw transaction. So we are going to use `bitcoin-cli send` in this example. The trick over here is that if you send only the hexdump of your message to the data section of the transaction, it becomes a `OP_RETURN` kind of tx.

```shell
# Fetching hexdump of my message
printf 'Sandipan<sandipndev> was here on 11th July, 2021' | hexdump | cut -c 9- | tr -d "\n "
53616e646970616e3c73616e6469706e6465763e207761732068657265206f6e2031317468204a756c792c2032303231

# Sending the same as a proof of burn tx
bitcoin-cli send '[{"data":"53616e646970616e3c73616e6469706e6465763e207761732068657265206f6e2031317468204a756c792c2032303231"}]'
{
  "txid": "350255af992426083f544e3dfdf4f0fbc1386f3ffc31b5ffa43059b26125987b",
  "complete": true
}

# Now let us look into the details of the raw tx
bitcoin-cli getrawtransaction 350255af992426083f544e3dfdf4f0fbc1386f3ffc31b5ffa43059b26125987b true
{
  "txid": "350255af992426083f544e3dfdf4f0fbc1386f3ffc31b5ffa43059b26125987b",
  "hash": "e5e1cf205d5fed884fabbc41c0c64d5b83fc7cdab2e76b5b45d06ebb9f27a458",
  "version": 2,
  "size": 398,
  "vsize": 236,
  "weight": 944,
  "locktime": 0,
  "vin": [
    {
      "txid": "dbc239cd636a8603c04530fd8c0349acd3b935d74bc2bdc0f8ef5ade75df9e95",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "30440220550c2d8ea9503bca67a42bc5d0220e3b9c0aff8ba3377af2572a70e38bf5c574022040a6b225f95e20261425db51879cc63bf9b0f1c06d7fef9ad6e6e519ffe2a3b101",
        "03efa5ae727e6251c9fe27c8c81c8260b5832dc7be6485e7ac85b6873a8de626cd"
      ],
      "sequence": 4294967294
    },
    {
      "txid": "1e839b8c33d9b1efcf2457bebb378aaa3e518d2b912d42944f36cf762c9ccbdf",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "304402206dd7e731b90bf1dd6cd5ff572e3a3b4d1719c29e4700b893155bd8d9158efde602201c66cf15ef8277e24b781d43d011f1d0f68d544029fdbcda0462f549232fa29f01",
        "03efa5ae727e6251c9fe27c8c81c8260b5832dc7be6485e7ac85b6873a8de626cd"
      ],
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.01999696,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 e5b7493fa05eda76ee053c62fb85fa0bdbdd3679",
        "hex": "0014e5b7493fa05eda76ee053c62fb85fa0bdbdd3679",
        "reqSigs": 1,
        "type": "witness_v0_keyhash",
        "addresses": [
          "tb1qukm5j0aqtmd8dms98330hp06p0da6dnet3ejkv"
        ]
      }
    },
    {
      "value": 0.00000000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_RETURN 53616e646970616e3c73616e6469706e6465763e207761732068657265206f6e2031317468204a756c792c2032303231",
        "hex": "6a3053616e646970616e3c73616e6469706e6465763e207761732068657265206f6e2031317468204a756c792c2032303231",
        "type": "nulldata"
      }
    }
  ],
  "hex": "02000000000102959edf75de5aeff8c0bdc24bd735b9d3ac49038cfd3045c003866a63cd39c2db0000000000feffffffdfcb9c2c76cf364f94422d912b8d513eaa8a37bbbe5724cfefb1d9338c9b831e0000000000feffffff0250831e0000000000160014e5b7493fa05eda76ee053c62fb85fa0bdbdd36790000000000000000326a3053616e646970616e3c73616e6469706e6465763e207761732068657265206f6e2031317468204a756c792c2032303231024730440220550c2d8ea9503bca67a42bc5d0220e3b9c0aff8ba3377af2572a70e38bf5c574022040a6b225f95e20261425db51879cc63bf9b0f1c06d7fef9ad6e6e519ffe2a3b1012103efa5ae727e6251c9fe27c8c81c8260b5832dc7be6485e7ac85b6873a8de626cd0247304402206dd7e731b90bf1dd6cd5ff572e3a3b4d1719c29e4700b893155bd8d9158efde602201c66cf15ef8277e24b781d43d011f1d0f68d544029fdbcda0462f549232fa29f012103efa5ae727e6251c9fe27c8c81c8260b5832dc7be6485e7ac85b6873a8de626cd00000000",
  "blockhash": "000000cf4e05eff5886b7018f2d51640c47566383f6818fcd6461475bf16feac",
  "confirmations": 1,
  "time": 1625994269,
  "blocktime": 1625994269
}

# You should notice that output 2 has OP_RETURN in the beginning.

# Lets find our string in the block data
strings ./bitcoin/.bitcoin/signet/blocks/blk*.dat | grep here
BitcoinCrashCourse was here
BitcoinCrashCourse was here
kalle was here
prady is here^DL
BitcoinCrashCourse was here
j
 aman is here
2j0Sandipan<sandipndev> was here on 11th July, 2021
```

Finally, we have our message embedded into the blockchain, forever!

## References

This post was written at [Summer of Bitcoin 2021](https://summerofbitcoin.org/?ref=blog.summerofbitcoin.org). All the exercises mentioned above were part of practice exercises that we had to complete during the course. Here is a list of references I used.

1. Kalle Rosenbaum's book - [Grokking Bitcoin](http://rosenbaum.se/book/?ref=blog.summerofbitcoin.org)
2. Nuances of Bitcoin CLI - [ChainQuery](https://chainquery.com/bitcoin-cli?ref=blog.summerofbitcoin.org)
3. Finding blocks and transactions on Signet - [Signet Explorer](https://explorer.bc-2.jp/?ref=blog.summerofbitcoin.org)
