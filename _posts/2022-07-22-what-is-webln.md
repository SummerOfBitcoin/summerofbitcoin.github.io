---
layout: post
title: What is WebLN?
author: Sanjay Singh Rajpoot
date: "2022-07-22 10:44:05 +0000"
categories:
  - "Tutorials"
  - "Lightning Network"
---

WebLN is a library and set of specifications for lightning apps and client providers to facilitate communication between apps and users’ lightning nodes in a secure way. It provides a programmatic, permissioned interface for letting applications ask users to send payments, generates invoices to receive payments, and much more. This documentation covers both how to use WebLN in your Lightning-driven applications and how to implement a provider.

Lightning Network is just another software layer, and we want to integrate it with the web, adding Lightning to web will go a long way in enabling bitcoin payments natively on the internet. This is precisely the idea behind [WebLN](https://webln.dev/?ref=blog.summerofbitcoin.org#/), which is a simple JavaScript tool to build Lightning-enabled browser extensions using makePayment and sendInvoice (again, the two core functions for any kind of money: sending and receiving).

WebLN offers a few advantages. First, JavaScript is nearly universal and almost thirty years old. Second WebLN delivers a better interface for the users, starting with the fact that you don’t need to *use a second device*. It feels native, not like a workaround. You also have access to all browser events, so a key press, a mouse click, a [scroll position](https://webln.twentyuno.net/scroll?ref=blog.summerofbitcoin.org), etc. can all trigger a payment.

# **Installation**

## **First Way: Install with Package Manager (Preferred)**

Install the `webln` library using your package manager of choice:

* npm: `npm install --save webln`
* yarn: `yarn add webln`

And import it into your project wherever you need it:

```js
import { requestProvider } from 'webln';
```

## **Second Way: Include Script (Alternative)**

Alternatively, you can include a script on your page that will load the library. **Be sure to keep the integrity check to prevent malicious Javascript from loading.**

```js
<script
  src="https://unpkg.com/webln@0.2.0/dist/webln.min.js"
  integrity="sha384-mTReBqbhPO7ljQeIoFaD1NYS2KiYMwFJhUNpdwLj+VIuhhjvHQlZ1XpwzAvd93nQ"
  crossorigin="anonymous"
></script>
```

Now the first that we need to do is to enable the WebLN provider in the web browser so that every component inside our website can use all the WebLN features. So in the next step, we will try to implement a WebLN provider with the help of which we will be able to use all the WebLN features.

Most of WebLN’s methods will prompt the user in some way, often times to make payments or have them provide the information they may feel is quite private. Before running `requestProvider` or other methods, make sure the user knows what your app does, and why they should allow your calls to run. Popping up as soon as they load a page will cause users to reject WebLN requests, or worse yet, bounce from your page.

# **`requestProvider`**

To begin interacting with a user’s Lightning node, you’ll first need to request a `WebLNProvider` from them. `WebLNProvider` is a class that various clients implement and attach to your web session. Calling `requestProvider` will retrieve the provider for you, and prompt the client for permission to use it. Once you get the provider, you're free to call all of the other API methods.


# **WebLN getInfo function**

Ask the user for some information about their node. The request may be rejected by the user depending on the provider implementation.

```js
function getInfo(): Promise<GetInfoResponse>;
```

Response

```js
interface GetInfoResponse = {
  node: {
    alias: string;
    pubkey: string;
    color?: string;
  };
}
```

# **`webln.makeInvoice`**

This is used to create an Invoice for the user by the app. This will return a BOLT-11 invoice. There are many ways to request an invoice they are

* By specifying an explicit `amount`, the user's provider should enforce that the user generate an invoice with a specific amount
* When an explicit `amount` is *not* set, the user can return an invoice that has no amount specified, allowing the payment maker to send any amount
* By specifying a `minimumAmount` and / or `maximumAmount`, the user's provider should enforce that the user generate an invoice with an amount field constrained by that amount

Amounts are denominated in satoshis. For large amounts, it’s recommended you use a big number library such as [bn.js](https://www.npmjs.com/package/bn.js?ref=blog.summerofbitcoin.org) as Javascript only supports 32 bit integers.

## **Parameters**

```js
function makeInvoice(args: RequestInvoiceArgs): Promise<RequestInvoiceResponse>;

interface RequestInvoiceArgs {
  amount?: string | number;
  defaultAmount?: string | number;
  minimumAmount?: string | number;
  maximumAmount?: string | number;
  defaultMemo?: string;
}
```

## **Response**

```js
interface RequestInvoiceResponse {
  paymentRequest: string;
}
```

# **What if there is an error?**

To handle this test case both apps and providers can make use of WebLN’s pre-defined errors. They can be found in `webln/lib/errors` and should be used when throwing and handling errors to best inform the user of what's going on:

And the provider should throw the correct error when possible, just add this code to your project and if there is any error that will be displayed directly on your browser:


## **Resources**

* [https://www.npmjs.com/package/webln](https://www.npmjs.com/package/webln?ref=blog.summerofbitcoin.org)
* [https://github.com/joule-labs/webln-docs](https://github.com/joule-labs/webln-docs?ref=blog.summerofbitcoin.org)
* [https://webln.dev/#/api/make-invoice](https://webln.dev/?ref=blog.summerofbitcoin.org#/api/make-invoice)
* [https://medium.com/@wbobeirne/making-a-lightning-web-app-part-4-c0997f4353b8](https://medium.com/@wbobeirne/making-a-lightning-web-app-part-4-c0997f4353b8?ref=blog.summerofbitcoin.org)
* [https://github.com/joule-labs/webln/issues/13](https://github.com/joule-labs/webln/issues/13?ref=blog.summerofbitcoin.org)
