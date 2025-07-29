---
layout: post
title: "My Summer of Bitcoin Journey: Implementing Taproot and Nested Segwit on a Hardware Wallet"
date: 2024-07-18
author: Bryan Elee
categories: [Cypherock, Tutorials]
image: ""
---

I got accepted to the summer of bitcoin in May, to work on implementing
taproot and nested segwit (address generation and transaction signing) support
for the Cypherock X1 Hardware wallet.

The onboarding process was standard. I was introduced to the team by
Cypherock’s product manager, Akshit, and placed on the embedded development
team.

My first task was assigned to me the next day, I was to develop a poc of
taproot and nested segwit (address generation and transaction signing) using C
language crypto library with very little documentation. I accepted the task
and made very naive projections of the timeline and this turned out to be a
hilarious mistake.

I got started…and was able to implement support for nested segwit in little
over a week and then got to work on taproot. Taproot is based on three Bitcoin
Improvement Proposals (BIPs):

  * Schnorr Signatures (BIP 340),
  * Taproot itself (BIP 341) and
  * Tapscript (BIP 342).

We are more interested in taproot key path spending, so the work involves
implementing BIP 340, BIP 341. My taproot implementation was built on the
secp256k1 library. The poc was completed in 5 weeks and I was tasked with
integrating the poc in the firmware and that’s where the problems began.

The secp256k1 library allocates it memory statically at compile time and this
result’s in a fairly large binary by embedded development standards. compiling
the firmware with secp256k1 resulted in a build that failed in link stage.

    
    
    ./lib/gcc/arm-none-eabi/10.3.1/../../../../arm-none-eabi/bin/ld: Cypherock-Main.elf section `.rodata' will not fit in region `FLASH'  
    ../lib/gcc/arm-none-eabi/10.3.1/../../../../arm-none-eabi/bin/ld: region `FLASH' overflowed by **60080** bytes  
    collect2: error: ld returned 1 exit status  
    ninja: build stopped: subcommand failed.

Changing `ecmult_window_size` in secp256k1 resulted in a modest binary size
reduction but couldn’t save the implementation.  
We are about 8 weeks into the project and there are about 4 weeks left for
summer of bitcoin. The next approach would be a standalone schnoor signature
implementation.  
The takeaway from this experience would be to ask for more information about
the hardware before implementation.
