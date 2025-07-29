---
layout: post
title: "Summer Of Bitcoin'23: Implementation of PsbtV2 in rust-bitcoin"
date: 2023-08-17
author: Subhradeep Chakraborty
categories: ['Stories', 'Rust']
---

Summer Of Bitcoin is a yearly summer internship program aimed for university
students to help them get into the bitcoin world as developers or designers.
This year, I got the opportunity to work as a SoB intern under the guidance of
Sanket Kanjalkar, a Cryptographic Engineer at Blockstream Research.

## The Project

In 2017, the BIP (Bitcoin Improvement Proposal) 174 was proposed by Andrew
Chow introducing a standard for PSBT (Partially Signed Bitcoin Transactions)
that allows a group of users to collaboratively sign a transaction. However,
the first version described in BIP 174 had a problem — No new inputs and
outputs could be added after a PSBT was created. Hence all users participating
in the transaction would need to send the inputs and outputs beforehand to the
one responsible for creating the PSBT. The creator would then need to transmit
the PSBT to others and they could independently sign it.

More recently, in 2021, the same author proposed a breaking update (BIP 370)
to the BIP 174 standard allowing users to independently add new inputs and
outputs to the created PSBT. This new standard is known as PSBT version 2
(PSBTV2). It introduces new fields like “Transaction Modification flags”,
“Fallback Locktime”, “Transaction Version” etc. and completely removes the
need of a previously created unsigned transaction.

The rust-bitcoin library, a rust implementation of various bitcoin network
protocol features, currently supports only the PSBTv0 i.e. the BIP 174. The
main goal of this SoB project is to implement the PSBTv2 standard without
deprecating PSBTv0.

It includes the following features —

  * Serialization/Deserialization of both PSBTv2 and PSBTv0.
  * Ability to add new inputs and outputs to PSBTv2.
  * Convert between PSBTv2 and PSBTv0.
  * Update existing APIs according to the new change.
  * Add the BIP 370 test vectors.

## Designing the API

The first task towards the project was to decide the new PSBT API design. Some
possible ways were —

  * **Adding new fields to the existing**`**Psbt**`**struct:** Consists of non-breaking changes. It simply adds new `Option` fields to the existing `Psbt` struct that supposed to exist when it is a version 2 transaction.  
The drawback is that in this case, even if the `version` field is set, say
`Version::PsbtV0`, there is no guarantee that the created `Psbt` is actually a
version 0 PSBT. It is completely up to the developers to check its validity
before performing any operations on it.

    
    
    // lib.rs  
    pub struct Psbt {  
        /// The unsigned transaction, scriptSigs and witnesses for each input must be empty.  
        pub unsigned_tx: Option<Transaction>,  
        /// The version number of this PSBT. If omitted, the version number is V0.  
        /// See https://github.com/rust-bitcoin/rust-bitcoin/pull/1218  
        pub version: Version,  
      
        // other fields  
        // ...  
      
        /// The corresponding key-value map for each input in the unsigned transaction.  
        pub inputs: Vec<Input>, // New Input, see below  
        /// The corresponding key-value map for each output in the unsigned transaction.  
        pub outputs: Vec<Output>, // New Output, see below  
      
        // New Psbtv2 Optional fields go here  
        /// 32-bit little endian signed integer representing the  
        /// version number of the transaction being created  
        pub tx_version: Option<i32>,  
        /// 32-bit little endian unsigned integer representing the transaction locktime  
        /// to use if no inputs specify a required locktime.  
        pub fallback_locktime: Option<LockTime>,  
        /// 8 bit unsigned integer as a bitfield for various transaction modification flags  
        pub tx_modifiable: Option<TxModifiable>, // or, Option<TxModifiable>  
    }  
      
    impl Psbt {  
        /// Serialize `Psbt`  
        pub fn serialize(&self) -> Vec<u8> {  
            match self.version {  
                Version::PsbtV0 => {  
                    // Still it is not guaranteed to be a version 0,  
                    // it may actually be a version 2 PSBT that  
                    // does not have the `unsigned_tx` field set.  
                }  
                _ => {}  
            }  
        }  
    }  
      
    // main.rs  
    fn main() {  
      let psbt = Psbt {  
      // ...  
      };  
      psbt.validate_version();  
      // Do further operations  
    }

  * **Introduction of**`**Psbt**`**and**`**PsbtInner**`**structs:** Consists of breaking changes. The above mentioned `Psbt` struct is renamed to `PsbtInner` and it is wrapped inside a `Psbt` struct. The whole point of this design is to guarantee the validity of the created PSBT according to the specified version and restrict its usage so that the above behavior is always maintained.  
While `PsbtInner` has all the fields public allowing developers to play with
them however they want, all the important functions (e.g. `serialize`,
`deserialize`, `combine`, `add_input` etc.) are exposed to `Psbt` only. The
advantage here is that these functions don’t need to validate the version
every time, they can simply assume it to be validated and proceed further.

    
    
    // lib.rs  
    pub struct PsbtInner {  
      // Similar to `Psbt` shown above  
    }  
      
    /// Internally stores and owns a `PsbtInner` instance.  
    /// A `Psbt` instance always guarantees that the underlying inner is  
    /// validated according to its version and hence is less error-prone.  
    pub struct Psbt {  
      inner: PsbtInner  
    }  
      
    impl Psbt {  
      /// The only constructor to create a `Psbt`  
      pub fn from_inner(inner: PsbtInner) -> Result<Psbt, Error> {  
         // Do all the validation checks  
      }  
      
      /// serialize `Psbt`  
      pub fn serialize(&self) -> Vec<u8> {  
          // No need to validate the version  
          // Simply believe the version and serialize accordingly  
      }  
      
      // other public facing functions  
    }  
      
    // main.rs  
    fn main() {  
        let psbt = PsbtInner {  
            // version 2 fields  
        };  
        let psbt = Psbt::from_inner(psbt);  
        // You can not serialize without creating the `Psbt` instance  
        let serialized = psbt.serialize();  
    }

Eventually we decided to follow the second pattern.

## Implementation

Here is the PR consisting of the code changes — Implement support for PsbtV2
by Subhra264 · Pull Request #1924 · rust-bitcoin/rust-bitcoin (github.com)

The first step was to introduce new fields to the existing structs (
`PsbtInner`, `Input`, `Output`) and modify the signatures of their existing
functions.

**Serialization/Deserialization**

Once the new fields were added, the major tasks were to support the
serialization and deserialization for both the versions of PSBT. The
`serialize` and `deserialize` functions are implemented for the `Psbt` struct.

**Version 2 functions**

Since BIP 370 introduces new features for PSBT, new functions are created to
implement those features. It includes functions like —

  * `Psbt::add_input(input: Input)` that takes an `Input`, checks if modifications to inputs are allowed and adds this input if its lock time is compatible with the rest of the existing inputs. Also, it ensures that the final computed lock time (using `Psbt::compute_locktime()`) does not change by adding the given new input if any of the existing inputs has a signature.
  * `Psbt::add_output(output: Output)` that takes an `Output` and adds it if output modification is allowed.
  * `Psbt::compute_locktime()` finds the final lock time by iterating over each `Input` and looking at its `required_time_locktime` and `required_height_locktime` fields. Returns `Error` if a valid lock time cannot be computed.
  * `Psbt::get_v0(self)` converts the `Psbt` into a new version 0 PSBT.
  * `Psbt::get_v2(self)` converts the `Psbt` into a new version 2 PSBT.

**Test Vectors**

The BIP 370 provides different test vectors to test the behavior of the
implementation and check if they are working as expected. The test vectors are
taken from here — bips/bip-0370.mediawiki at master · bitcoin/bips
(github.com).

## My Learnings

Being a newbie in Rust, this project helped me get a deeper understanding of
the language and its various design patterns. The project helped me have a
practical experience of different rust concepts like borrowings, lifetimes,
macros, ownerships, and many more.

I am new to the Bitcoin world as well and hence this summer was extremely
delightful to get the opportunity to understand the CS and economics involved
— cryptography, p2p, time synchronization, blocks, transactions, etc.

## What Next?

The immediate next step will be to implement the new API changes in the rust-
miniscript as soon as they are merged into the rust-bitcoin library. I will
continue to contribute to the Bitcoin community and hopefully can add great
value to it.
