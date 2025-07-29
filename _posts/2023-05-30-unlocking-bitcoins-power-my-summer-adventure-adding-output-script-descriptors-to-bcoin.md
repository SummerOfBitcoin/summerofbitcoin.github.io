---
layout: post
title: "Unlocking Bitcoin's Power: My Summer Adventure Adding Output Script Descriptors to Bcoin"
date: 2023-05-30
author: Vasu Maheshwari
categories: ['Stories', 'Bitcoin Core']
---


<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*4Z79Ygu7GYOGL5MTXDMSjg.png"/>
</figure>

During this summer, I was fortunate to engage in the ‘S** _upport for output
script descriptors’_** project within the[**Bcoin**](https://github.com/bcoin-
org/bcoin)organization. Participating in the **Summer of Bitcoin** initiative
proved to be a profoundly enriching journey, offering a significant learning
curve within the realm of **Bitcoin** technology. I’d like to express my
sincere gratitude to [**Adi Shankara**](https://twitter.com/adi_shankara_) for
expertly guiding this year’s Summer of Bitcoin program. _It was indeed a
transformative journey._

> As I delved further into the intricacies of Bitcoin, I found myself
> captivated by the remarkable depth of its underlying technology.

Throughout the duration of the project, I had the privilege of working under
the guidance of accomplished and fervent experts, namely [**Matthew
Zipkin**](https://twitter.com/MatthewZipkin) and [**Anmol
Sharma**](https://twitter.com/theanmolsharma_). Their mentorship not only
expanded my knowledge but also played a pivotal role in the successful
realization of the project. It is imperative to acknowledge that without their
unwavering support, the fruition of this endeavor would have remained
unattainable.

# **_What are Descriptors?_**

>
> wsh(multi(2,03a0434d9e47f3c86235477c7b1ae6ae5d3442d49b1943c2b752a68e2a47e247c7,03774ae7f858a9411e5ef4246b70c65aac5649980be5c17891bbec17895da008cb,03d01115d548e7561b15c38f004d734633687cf4419620095bc5b0f47070afe85a))#en3tu306

An **Output Descriptor** is a string of characters in human-readable format
that encodes an output script, including all necessary information to solve
the script.

You can also think of descriptors as functions. A descriptor returns a script.
Some descriptors may incorporate a checksum (part after **#**) similar to
bech32 format, providing an additional layer of error detection to minimize
the risk of losing or incorrectly typing characters in the descriptor string
when shared with others. We can demonstrate the structure of a descriptor in
pseudo code as:

    
    
    descriptor_name(argument) => script.

A descriptor wallet is one which stores output descriptors and uses them to
create addresses and sign transactions. By abstracting away address creation
and transaction signing to a largely standalone module, such wallets can
upgrade to using new address types much more easily.  
For in depth study on what arguments are allowed inside descriptor and types
of descriptors must refer [**BIP
380**](https://github.com/bitcoin/bips/blob/master/bip-0380.mediawiki)**** and
this
[**doc**](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md).

> **What is the power of Descriptors?** Their foremost utility lies in their
> facilitation of the seamless import and export of seeds and keys, thereby
> enabling effortless transitions between distinct wallet implementations.
> They also allow you to build up the precise sort of addresses that you’re
> interested in creating.

# **What changes are being introduced to Bcoin with the implementation of the
Descriptor module?**

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NJCBiAboUXdHFftnNg9yGw.png"/>
</figure>

> **Bcoin** is**** an advanced full node alternative implementation of the
> Bitcoin protocol, developed using a blend of **JavaScript** and **C/C++** ,
> specifically designed to operate within the **Node.js** runtime environment.
> It boasts comprehensive testing and a profound understanding of all
> established bitcoin consensus regulations. Moreover, it also functions as a
> versatile **bitcoin library** suitable in Node.js environments and web
> browsers.

The integration of the Descriptor module into Bcoin introduces a pair of novel
RPCs: getdescriptorinfo and deriveaddresses. Along with it createmultisig rpc
is also redefined with the descriptor functionality.

> **_getdescriptorinfo_** — This is one of the most important rpc’s which is
> used for analysing the descriptor. It takes as input a descriptor field and
> outputs the information regarding the input descriptor along with the
> checksum.
    
    
    {                                   (json object)  
      "descriptor" : "str",             (string) The descriptor in canonical form, without private keys  
      "checksum" : "str",               (string) The checksum for the input descriptor  
      "isrange" : true|false,           (boolean) Whether the descriptor is ranged  
      "issolvable" : true|false,        (boolean) Whether the descriptor is solvable  
      "hasprivatekeys" : true|false     (boolean) Whether the input descriptor contained at least one private key  
    }

Source —
[_https://developer.bitcoin.org/reference/rpc/getdescriptorinfo.html_](https://developer.bitcoin.org/reference/rpc/getdescriptorinfo.html)

If you seek insight into the implementation details of the getdescriptorinfo
RPC within Bcoin, please consult this[**merged pr**](https://github.com/bcoin-
org/bcoin/pull/1152) referenced here.

> **_deriveaddresses_** — Again a very important rpc which is used for
> deriving addresses the descriptor can produce.
    
    
    [           (json array)  
      "str",    (string) the derived addresses  
      ...  
    ]

Source —
__[_https://developer.bitcoin.org/reference/rpc/deriveaddresses.html_](https://developer.bitcoin.org/reference/rpc/deriveaddresses.html)

If you seek insight into the implementation details of the deriveaddresses RPC
within Bcoin, please consult this****[**merged pr**
](https://github.com/bcoin-org/bcoin/pull/1162)referenced here.

> **_createmultisig_** — Used for creating multi-signature address with n
> signature of m keys required.
    
    
    {                            (json object)  
      "address" : "str",         (string) The value of the new multisig address.  
      "redeemScript" : "hex",    (string) The string value of the hex-encoded redemption script.  
      "descriptor" : "str"       (string) The descriptor for this multisig  
    }

Source —
__[_https://developer.bitcoin.org/reference/rpc/createmultisig.html_](https://developer.bitcoin.org/reference/rpc/createmultisig.html)

If you seek insight into the implementation details of the createmultisig RPC
within Bcoin, please consult this****[**open pr**](https://github.com/bcoin-
org/bcoin/pull/1171)referenced here.

# Overall Learnings and Experience

In conclusion, my journey into the realm of open-source contribution has
profoundly transformed my coding approach. Embracing and adhering to the
meticulous rules and practices of the organization has instilled a newfound
discipline in my coding style. The experience has truly broadened my technical
proficiency.

> A significant milestone achieved during this journey was conquering my
> apprehension of version control systems, particularly Git, as I now navigate
> its functionalities with confidence.

The emphasis on effective communication within the community has proven
invaluable, reinforcing the notion that coding is as much about collaboration
as it is about individual skill.

This program has provided me with an invaluable opportunity to delve deeper
into the intricate world of the Bitcoin rabbit hole, further stoking my
curiosity and passion for the technology.

> **As I reflect on this transformative experience, I am excited about the
> prospects of continued learning, collaboration, and meaningful contributions
> in Bitcoin ecosystem.**

Thanks for reading!

