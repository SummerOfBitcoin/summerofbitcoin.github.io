---
layout: post
title: Making Bitcoin Transactions Smart with Structured Metadata
author: Summer of Bitcoin
date: "2022-07-22 12:22:30 +0000"
tags:
  - "Tutorials"
---

**By Pavan Joshi   
Summer of Bitcoin 2022**

## Abstract ⚡

Specifications such as WebLN have brought better solutions to the UX for the lightning network.

Many broad applications are now possible, such as 💻 —

* Instant payments via a web browser using Bitcoin Sats (eg. Alby)
* Tippings for content writers, podcasts and streamers
* Streaming sats
* Instant purchases on the Web

Transaction lists as we know them from our private bank accounts are often a simple list of transactions sorted by date. Each transaction has data like Sender, Receiver, amount, reason and date.

Similarly, Bitcoin transactions hold only static data which doesn’t include much more relevant information to look into.

What if transactions can have additional information (metadata) such as 💫 —

* Purpose of transaction
* What was bought with that particular transaction
* Accessing what was bought right out of the wallet
* Allowing users to interact with transactions depending on the additional information that transactions hold

Every transaction can store additional information in form of metadata which can make specifications such as WebLN more broad and interactive.

This project aims to extend WebLN to enrich transactions with additional information aka structured metadata so that such transactions gain additional interactivity.

## What is WebLN? 🤔

WebLN is a set of specifications for Lightning apps and client providers to facilitate communication between web apps and users’ lightning nodes in a secure way. It provides a programmatic, permissioned interface for letting applications ask users to send payments, generate invoices to receive payments, and much more.

![WebLN Provider Functionality](https://cdn-images-1.medium.com/max/3530/0*8D4j3sWnsbclRJga)

## Sending Payments Via WebLN Along With Metadata🧾

WebLN ***SendPayment*** takes ***paymentRequest*** parameter holding Bolt11 invoices. We can extend this function to add an extra “optional” parameter named ***metadata*** which will hold metadata as a string that can be passed on to the wallets.

***Function Signature::***

```
sendPayment(paymentRequest: string, metadata?: string): Promise<SendPaymentResponse>;

```

**WebLN Provider attached by wallets currently**

<figure>
<img src="https://cdn-images-1.medium.com/max/2478/0*xH3Ip4lBH_1Xzj0S.png"/>
<figcaption>**WebLN Provider attached by wallets after implementation of spec**</figcaption>
</figure>

<figure>
<img src="https://cdn-images-1.medium.com/max/2000/0*ck3K-T5O2haHBVqy.png"/>
<figcaption>## Working/architecture of this spec ⚙️</figcaption>
</figure>

<figure>
<img src="https://cdn-images-1.medium.com/max/3694/1*Lz4JHjDty-jrWAV9eqcDPA.png"/>
<figcaption>1. **Make a Request to Wallet for WebLN Provider Via Triggering SendPayment()**</figcaption>
</figure>

   `sendPayment(paymentRequest: string, metadata?: string): Promise<SendPaymentResponse>;`
2. **Structure Metadata and pass it to the wallet in Callback**

   Use [**Schema.org**](http://schema.org/?ref=blog.summerofbitcoin.org) specifications to structure metadata in form of **JSON-LD.**

   eg.

   ```
   var Metadata = {};
   Metadata = {
    "type": "AudioObject",
    "name": "title",
    "creator": "artist",
    "image": "image" 
    }
    export var Metadata;

   ```

   Learn more about how to structure metadata for bitcoin transactions [here](https://github.com/getAlby/lightning-browser-extension/wiki/Structuring-Transaction-Metadata-Using-Schema.org-Specifications?ref=blog.summerofbitcoin.org).
3. **Validate Metadata**  
   Since metadata is user-generated, it should be cross-checked or validated for its correctness.

   *Enter JSON Schema* 🧐

   JSON Schema is **a JSON media type for defining the structure of JSON data**. JSON Schema provides a contract for what JSON data is required for a given application and how to interact with it.

   Schemas can be created to validate received data against a predefined data model. To do this, many validator plugins are available.

   * Generate Schema
   * Compile Schema
   * Validate received data for that particular schema

   [Zod](https://www.npmjs.com/package/zod?ref=blog.summerofbitcoin.org) plugin can be used to validate JSON schemas as well as object schemas. The best thing about this plugin is that it has zero external dependencies involved.

   **Validator function which validates schemas using zod plugin**

   * Schema Created using Zod plugin

   ```

    import { z } from "zod";

    export const audioObjectSchema = z.object({
      type: z.string(),
      name: z.string().optional(),
      creator: z.string().optional(),
      image: z.string().optional(),
    });

   ```

   * Validator function to validate metadata

   ```

    import { audioObjectSchema } from "./audioObjectSchema";

    export function isBase64(str: string) {
      if (str === "" || str.trim() === "") {
        return false;
      }
      try {
        return btoa(atob(str)) == str;
      } catch (err) {
        return false;
      }
    }

    export function MetadataValidator(metadata: { [key: string]: string }) {
      let hasValidType = false;
      let hasValidImage = true;
      for (const key in metadata) {
        if (key == "type") {
          if (metadata[key] == "AudioObject") {
            const parsed = audioObjectSchema.safeParse(metadata);
            if (parsed.success) {
              hasValidType = true;
            }
          }
        }
        if (key == "image") {
          hasValidImage = isBase64(metadata[key]);
        }
      }

    return hasValidType && hasValidImage;
    }

   ```

   * To validate metadata, just call the function by passing metadata.

   ```
   const isMetadataValid = MetadataValidator(metadata);

   ```
4. **Render Metadata in Confirmation Dialogue (if valid)** 🚦

   If metadata is successfully validated, we can show the user the amount they need to pay as well as what it is they are paying for.

   In short, users can see the purpose of the transaction and the amount they need to pay, before making the actual payment for it.
5. **Store, interact and perform actions with metadata** 💾

   Once the user confirms his/her payment and the transaction gets successful -

   * In Bitcoin Lightning Wallet

   We can store and render the metadata in the transaction database of the wallet allowing users to interact with the metadata.

   * In Lightning Application

   Once the payment is successful , we can allow the user to access their purchased content such as allowing the user to download a song.

   ```
    webln.sendPayment(Bolt 11 invoice, Metadata)
             .then(function(r) {
               if(r != undefined){  
               // do after payment actions with the metadata. eg. allowing user to download song after payment is done 
             }
             })
             .catch(function(e) {
               alert("Failed: " + e.message);
               console.log('err pay:', e);
             });
       })
       .catch(function(e) {
         alert("Webln error, check console");
         console.log('err, provider', e);
       });
   }

   ```

### Usecases 🔥

The potential is huge. Transactions no longer just hold details about the payment but also include structured metadata of the consumed objects. 💫

For instance, it allows users to perform actions directly out of their wallets such as accessing purchased digital content, renewing expiring subscriptions, recollaterizing open trades on trading platforms or topping up gift card budgets. ✨

To demonstrate concept more clearly, I decided to create a simple prototype. A visitor on a music website can check out a song, and buy it if they like it. On the payment confirmation page, they can see the song name, artist, and its cover image (sent as base64 encoded string, decoded by Alby).

After payment is successful, the song gets downloaded into users’ local storage. 🎉

Check out the demo video of the entire flow [here](https://drive.google.com/file/d/1WUZybY3d-MTbcdlMRGQK_zTJHU7PVeIV/view?ref=blog.summerofbitcoin.org).

[Here](https://github.com/pavanjoshi914/Buy-songs-with-Alby-demo-for-transaction-metadata/pull/1?ref=blog.summerofbitcoin.org) is the source code for the prototype.

[Here](https://github.com/pavanjoshi914/Alby-With-extended-Webln/commit/ed6f7559de47b7a9c932e655fb45be9481856714?ref=blog.summerofbitcoin.org) is the spec for extending WebLN with metadata.