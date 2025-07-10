---
layout: post
title: Running Bitcoin Core, Signet and playing with test bitcoins using Bitcoin-CLI 🌠
author: Priyansh Rastogi
date: "2022-03-29 15:17:00 +0000"
categories:
  - "Tutorials"
---

# Installation ⬇️

The installation steps are for a Linux device, much of it should apply for Windows and MacOS. One can download the latest version of bitcoin core from [bitcoincore.org](https://bitcoin.org/en/bitcoin-core/?ref=blog.summerofbitcoin.org).  
After downloading, you need to check if the file you downloaded is the original one or not by verifying the hash of the file.  
You can use the following commands for downloading and verifying:

```shell
#get the file
$ wget https://bitcoincore.org/bin/bitcoin-core-0.21.1/bitcoin-0.21.1-x86_64-linux-gnu.tar.gz

#get the signature file
$ wget https://bitcoincore.org/bin/bitcoin-core-0.21.1/SHA256SUMS.asc

#verify that the hash of the tar.gz file matches the one in SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS.asc
```

Now we need to check if the signature is correct or not. You need to lookup the fingerprint of Bitcoin Core’s release key.  
It should be 01EA5486DE18A882D4C2684590C8019E36C2E964  
but you should verify it from multiple places like community channels, twitter, bitcoincore.org

```shell
$ gpg --keyserver hkp://keyserver.ubuntu.com --recv-keys 01EA5486DE18A882D4C2684590C8019E36C2E964

#verify the signature file
$ gpg --verify SHA256SUMS.asc
```

![Screenshot from 2021-07-12 20-43-07.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626102804267/aRi2fBbZI.png)

Be sure to get such output and also crosscheck the fingerprint.

Yes, we need to perform some security checks while downloading softwares related to bitcoin. Hackers love Bitcoin a lot. 🧐

# Getting Started with Bitcoin Core ✨

First you need to unpack the tar.gz file.

```shell
#unpacking 
$ tar -zxvf bitcoin-0.21.1-x86_64-linux-gnu.tar.gz

#switch directory 
$ cd bitcoin-0.21.1/bin
```

Start bitcoin core in the background on signet test network. Since signet is a test network, it would take a few minutes to sync the entire blockchain data.

```shell
$ ./bitcoind -signet -daemon

#look at blockchain data
$ ./bitcoin-cli -signet getblockchaininfo

#stop bitcoin core
$ ./bitcoin-cli -signet stop
```

Lets get rid of writing signet in every command by adding it in bitcoin configuration file.

```shell
$ echo “chain=signet” >> ~/.bitcoin/bitcoin.conf

#enable transactions index 
$ echo “txindex=1” >> ~/.bitcoin/bitcoin.conf

#starting bitcoin core without signet flag
$ ./bitcoind -daemon
```

Be sure to perform all the above commands in /bitcoin-0.21.1/bin/ directory.  
Else it will give an error.

![Screenshot from 2021-07-12 20-57-37.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626103678277/5iPhqSoDe.png)

Woohoo!!  
Your bitcoin core is up and running.

# Create a wallet 👜

Let's create a bitcoin wallet and play with some test bitcoins.

```shell
$ ./bitcoin-cli -stdin createwallet my false false
Enter the password and hit ctrl+d

$ ./bitcoin-cli getwalletinfo
```

Let me explain -

* -stdin: prompts user for password
* createwallet: creates a new wallet
* my: wallet name

The next 2 parameters need to be put as false for now, they hold special meaning.  
If you are curious use the help command to know more:

```shell
#you can use help to get more info of any command
./bitcoin-cli help createwallet
```

![Screenshot from 2021-07-12 21-09-27.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626104554213/3XyhVunmp.png)

Once the wallet is created you can get the wallet info.

![Screenshot from 2021-07-12 21-13-19.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626104613803/TzXkhvyaA.png)

Since Bitcoin Core doesn't use mnemonic phrase you may need to backup your wallet.

```shell
$ ./bitcoin-cli backupwallet ~/walletbackup.dat
```

# Get some test bitcoins 💰

First you need to create a new address in the wallet to store some bitcoins.  
You can use [Signet Faucet](https://signet.bc-2.jp/?ref=blog.summerofbitcoin.org) to get some test coins. Paste the output of getnewaddress in the website and get 0.1 BTC (Remember test Coins😂)

```shell
#create new address
 ./bitcoin-cli getnewaddress

#get confirmed balance
 ./bitcoin-cli getbalance

#get confirmed and unconfirmed balance
 ./bitcoin-cli getbalances
```

# Send Coins 💸

The private key of your wallet is encrypted, so in order to use it you need to decrypt it for a certain time span.

```shell
./bitcoin-cli -stdin walletpassphrase
Enter the password
[enter time in seconds for which you want your private key to stay decrypted. eg: 500000]

#send coins 
$ ./bitcoin-cli -named sendtoaddress address='receiver's address' amount=1
```

You did it. You just sent some bitcoins to an address! 💪
