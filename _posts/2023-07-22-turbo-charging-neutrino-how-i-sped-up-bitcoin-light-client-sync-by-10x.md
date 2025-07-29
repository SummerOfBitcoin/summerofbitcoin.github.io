---
layout: post
title: "Turbo-Charging Neutrino: How I Sped Up Bitcoin Light Client Sync by 10x"
date: 2023-07-22
author: Maureen Ononiwu
categories: ['Stories', 'Lightning']
---

## **INTRODUCTION**

Neutrino is a privacy-preserving light client. It is "light" as it does not download the entire Bitcoin block. Rather, it downloads only Bitcoin block headers and block filter headers. This makes it, "light" as the average size of a Bitcoin block is 3-3.5 MB while a Bitcoin block header is eighty bytes and a filter header is 32 bytes only. It updates its header store and filter store by syncing in a peer-to-peer connection with full nodes. To improve the speed at which neutrino updates its header, the community has decided to [implement sideloading](https://github.com/lightninglabs/neutrino/issues/70).

Sideloading in this context involves the process of fetching headers from a local source instead of over the internet in the normal peer-to-peer connection.

## IMPLEMENTATION

To implement the sideloading functionality, I created a new Golang interface that would represent the interface the block manager would interact with as the reader. Below is the definition of the interface

```go
type Reader interface {
	NextHeader() (*wire.BlockHeader, error)

	EndHeight() int64

	StartHeight() int64

	SetHeight(int64) error

	HeadersChain() wire.BitcoinNet
}
```

**NextHeader method**

The NextHeader method is used to fetch headers from the sideload source.

**EndHeight method**  
The EndHeight method returns the height of the last header in the sideload source.

**StartHeight method**  
The StartHeight method returns the height of the first header in the sideload source

**SetHeight method**

The SetHeight method takes in a height variable which represents the height of the last header the caller (the block manager in this case) has in its store as an argument then computes the required offset required to fetch the next header after the height variable in the file. In summary, the setHeight method calculates the required distance one has to seek to fetch the next header after the height variable.

**HeadersChain method**

The headersChain method returns the network chain (i.e. testnet, mainnet, simnet, and regtest) which the header belongs to.

A sideload source could be a binary file, or hex-encoded file to mention a few. All sideload sources must create a structure that implements this interface to enable interaction with Neutrino's block manager

## Binary file implementation of the reader.

Binary files are what is currently used in Neutrino to store headers. I created an implementation of the reader interface for binary files. My mentor, [Jordi Montes](https://twitter.com/positiveblue2) came up with the idea to enforce an encoding for all binary files to be used as a sideload source. The encoding involves using the first 161 bytes of a file to store descriptive data about the headers contained in the file. The encoding is as follows:

1. The first 80 bytes should contain the height of the first header in the file known this would be known as the `startHeight`.
    
2. The following 80 bytes after the area representing the `startHeight` should contain the height of the last header in the binary file, this would be known as the `endHeight`
    
3. The following one byte after the area containing the end height should contain characters, r, t, m, or s. These characters represent the chain the headers contained in the file belong to.
    
    *r represents the regtest network, t represents the testnet network, m represents the mainnet network, s represents the simnet network.*
    

If sideloading is enabled in the chain, the binary file is opened and the start Height, end height and network of the header file is read and stored in a struct to be accessed by the Reader methods when needed. The open file is also stored in a struct. The `nextHeader` function reads from this file to supply headers to the caller in this case the blockmanager.

To take a look at the code go through this [pull request](https://github.com/lightninglabs/neutrino/pull/285)

## Acknowledgment

This project is part of my [Summer of Bitcoin](http://summerofbitcoin.org) project and I undertook it under the mentorship of [Jordi Montes](https://twitter.com/positiveblue2). He came up with a lot of the brilliant ideas involved in completing this project and I would like to thank him for his patience and guidance throughout this process.
