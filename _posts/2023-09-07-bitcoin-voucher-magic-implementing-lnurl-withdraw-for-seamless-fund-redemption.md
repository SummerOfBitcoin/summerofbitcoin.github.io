---
layout: post
title: "Creating Vouchers with Bitcoin: Implementing LNURL Withdraw"
date: 2023-09-07
author: Siddharth Tiwari
categories: ['Stories', 'Wallets']
---


 _Before Going through this, I would recommend you first read_ _this_ _article
to get a basic understanding of Lighting Networks and LNURL 👉_ _Click here_

> **A little Intro  
> ** Cryptocurrencies have revolutionized the world of finance, offering
> secure, decentralized, and fast transactions. Bitcoin, the leading digital
> currency, has seen significant advancements, including the Lightning Network
> (LN), which enables instantaneous micro-transactions with reduced fees. In
> this blog post, we will explore the concept of LNURL withdraw and guide you
> through the technical process of creating vouchers to allow users to
> withdraw their Bitcoin funds. Let’s dive in! 🚀💡

# **Alright, let’s break it down!**

We’ve got three main players in this voucher game: the voucher creator, the
voucher redeemer, and the escrow wallet or account. 🎮💰

The voucher creator is the one who makes things happen. They whip up a cool
voucher by setting the amount and sending the funds to the escrow account.
It’s like creating a special treat for someone! 🍭✨

Now, the voucher redeemer is the lucky person who gets to claim the voucher
and enjoy the rewards. They’re excited to redeem the voucher and receive the
specified amount. It’s like winning a prize! 🎁🎉

But hold on, what’s with this escrow account? Well, think of it as the
trustworthy middleman who holds the voucher amount and takes care of the
funds. It’s like having a cool friend who keeps things safe until you’re ready
to claim your prize. 🕶️🔒

Instead of directly creating a voucher and having the redeemer claim it, we
involve the escrow account to make things easier. The escrow account ensures
that there’s enough money available when the redeemer wants to cash in their
voucher. It’s like having your own personal money manager! 💼💸 This allows them
to interact seamlessly with the application. Otherwise, they would need to
share their wallet access tokens to send money to the redeemer when they wish
to withdraw the funds, and users with different wallets can use the
application.

To facilitate the voucher creation and redemption process, users need to have
a Lightning Network (LN) wallet that supports the LNURL protocol.

So, who can act as the escrow? In this case, we can utilize a Bitcoin banking
management system or a wallet organization with API support. One recommended
option is **Galoy** , an open-source Bitcoin banking infrastructure system. It
offers a very well-documented GraphQL API, which is actively maintained and
has extremely low transaction fees for Lightning transactions. **Galoy** also
provides a testing and playground API where you can thoroughly test your
application. The most interesting feature of **Galoy** is**“stable sats”**
which refers to Satoshi that maintains a stable value, similar to fiat
currency. You can find more information about stable sats at stablesats.com.
🏦⚙️

# How do these three interact?

Now, let’s discuss how the application would work with these three characters:
the voucher creator, the voucher redeemer, and the escrow account. 📝🧾

The voucher creator initiates the process by entering the desired amount and
sending funds to the escrow account. If you’re using Galoy, you can use the
GraphQL query `**lnUsdInvoiceCreateOnBehalfOfRecipient** ` to create an
invoice. Once the invoice is successfully created, the user funds it by
sending the specified amount. 💸💼 It’s like filling up a piggy bank! 🐷💰

After the successful execution of the above steps, the application backend
saves the necessary data, such as the amount in “**satoshis** ” or “**cents**
” if you’re using stable sats.  
You can even get creative and add features, by adding options such as
expiration dates, limited usage, PIN codes for access, and more. 📊📅🔢

# LNURL withdraw.

Now comes the fun part: LNURL withdrawal! The redeemer user receives an LNURL
string, which looks like a long code. This LNURL string encodes the URL of
your API. When a supported LN wallet accesses this LNURL string by either
scanning a QR code or directly inputting it, the wallet hits the encoded URL
and sees the options available. 📲🔗

This is what **LNURL strings** look like.

    
    
    LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS

When decoded, this string will look like a normal URL like this.

**_https://service.com/api.…._**

So when a supported LN wallet access the LNURL string by directly passing or
scanning with QR, the wallet will hit the decoded URL with a GET request and
this is what the wallet will see.  
For LNURL withdrawal, the response will look like this.

    
    
    {  
    "tag": "withdrawRequest", // type of LNURL  
    "callback": string, // The URL which LN SERVICE would accept a withdrawal Lightning invoice as query parameter  
    "k1": string, // Random or non-random string to identify the user's LN WALLET when using the callback URL  
    "defaultDescription": string, // A default withdrawal invoice description  
    "minWithdrawable": number, // Min amount (in millisatoshis) the user can withdraw from LN SERVICE, or 0  
    "maxWithdrawable": number, // Max amount (in millisatoshis) the user can withdraw from LN SERVICE, or equal to minWithdrawable if the user has no choice over the amounts  
    }

Now, The wallet allows the redeemer to choose the amount they want to withdraw
within the given range (if options are provided). Once accepted by the user,
LN wallets send a GET Request to the application in the form of.

`<callback> <?|&> // either '?' or '&' depending on whether there is a query
string already in the callback k1=<k1> // the k1 specified in the response
above &pr=<lightning invoice> // the payment request generated by the wallet`

So basically, Wallet creates an invoice for the amount that the user wants to
withdraw and sends it with a GET Request callback URL.

Now when the application/backend receives the request, it checks various
parameters, such as the voucher’s payment status, and other conditions, and if
everything is in order, it proceeds to pay the invoice. The response sent to
the redeemer’s wallet will be****`{“status”: “OK”}`, indicating that the funds
are on their way. 🔄💰

In case of any errors or issues, the wallet receives a response in the format
`{"status": "ERROR", "reason": "error details…"}`. The wallet then alerts the
user with an appropriate message, ensuring they stay informed about the status
of the transaction. ❌🔔

By implementing the above workflow, you can create vouchers with Bitcoin using
LNURL withdrawals. This user-friendly and efficient approach enhances the
usability and accessibility of Bitcoin transactions, making it easier for
users to withdraw their funds. So, why wait? Start exploring LNURL withdrawal
and provide your users with a seamless and secure way to redeem their Bitcoin
vouchers! 🎉💰

For more detailed information on LNURL withdrawal and how it works, you can
check out the following resources:

**LNURL Withdraw Specification:** If you’re curious about the technical
details and want to dive deeper into the inner workings of LNURL withdrawal,
you can visit the LNURL GitHub repository and specifically explore the
luds/03.md file. Here’s the link: **LNURL Withdraw Specification**

**Galoy Documentation:** If you’re interested in using Galoy as your Bitcoin
banking system, they’ve got some fantastic documentation that can guide you
through the implementation process. You can find it on their website at
**Galoy Documentation**.

So, grab a cup of coffee, sit back, and enjoy exploring these resources.
They’ll provide you with all the juicy details and technical know-how you need
to create an awesome voucher system with LNURL withdrawal. Happy coding! 💻☕️
