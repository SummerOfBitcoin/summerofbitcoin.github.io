---
layout: post
title: "Building the Receive Functionality for BIP21 URIs in LDK-Node: A Technical Deep Dive"
date: 2024-08-12
author: "Ian Slane"
categories: [Tutorials, LDK]
---

In this article, we discuss one of the main aspects of my Summer of Bitcoin project: implementing the receive functionality. This involves generating URIs that can handle multiple payment scenarios, including BOLT11 invoices and BOLT12 offers. In this post, I’ll take you through the initial rough draft, a step-by-step guide on the implementation, and code snippets with explanations. I’ll also show the updates to the serialize and deserialize logic from the last post.

### IMPLEMENTING THE RECEIVE FUNCTIONALITY

#### UNIFIEDQRPAYMENT HANDLER

The UnifedQrPayment handler plays a crucial role in the overall implementation. Here a breakdown of its structure and functionality:

```rust
pub struct UnifiedQrPayment {
    onchain_payment: Arc<OnchainPayment>,
    bolt11_invoice: Arc<Bolt11Payment>,
    bolt12_payment: Arc<Bolt12Payment>,
    logger: Arc<FilesystemLogger>,
}
```

The `UnifiedQrPayment` handler struct integrates three key components: `onchain_payment`, `bolt11_invoice`, and `bolt12_payment`. These components are responsible for generating the necessary payment details—on-chain address, BOLT11 invoice, and BOLT12 offer—that are included in the URI.

Fortunately, I was able to utilize these existing payment handler APIs for these payment options, which streamlined the integration process. The components are private within the `UnifiedQrPayment` struct, ensuring that they cannot be directly modified by the user. Instead, they are managed internally, preventing alterations and ensuring that the payment details are generated accurately and securely.

#### IMPLEMENTING THE RECEIVE FUNCTION

Now, let’s dive into the core of the matter: the `receive` function. This function generates a URI that includes an on-chain address, a BOLT11 invoice, and a BOLT12 offer. Here's a breakdown of its implementation:

```rust
impl UnifiedQrPayment {
    pub(crate) fn new(
        onchain_payment: Arc<OnchainPayment>,
        bolt11_invoice: Arc<Bolt11Payment>,
        bolt12_payment: Arc<Bolt12Payment>,
        logger: Arc<FilesystemLogger>,
    ) -> Self {
        Self { onchain_payment, bolt11_invoice, bolt12_payment, logger }
    }

    pub fn receive(
        &self, amount_sats: u64, message: &str, expiry_sec: u32,
    ) -> Result<String, Error> {
        // Function implementation
    }
}
```

The `new` function serves as a constructor that initializes a new instance of `UnifiedQrPayment` (which is itself initialized as an instance of Node, to be discussed later) which is not public to the user. This function sets up the provided payment handlers along with a logger to give better error reasoning.

The `receive` function requires three parameters: the amount to receive (in sats), a payment message visible to the sender, and an HTLC expiration time. If successful, the function returns a URI as a String; otherwise, it returns an Error.

#### FUNCTION OVERVIEW

The function starts by generating a new on-chain address using the `onchain_payment` handler. The `new_address()` method from the on-chain payment handler generates the address returned as a String or an Error.

```rust
let onchain_address = self.onchain_payment.new_address()?;
```

Next, we convert the provided amount in satoshis to millisatoshis, the unit used by lightning payments.

```rust
let amount_msats = amount_sats * 1_000;
```

We then attempt to generate a BOLT12 offer using the `bolt12_payment` handler. If successful, the offer is included; otherwise, an error is logged. The `bolt12_payment` handlers receive function creates the offer and returns the offer as a String or Error much like our receive function.

```rust
let bolt12_offer = match self.bolt12_payment.receive(amount_msats, message) {
    Ok(offer) => Some(offer),
    Err(e) => {
        log_error!(self.logger, "Failed to create offer: {}", e);
        None
    },
};
```

Similarly, we attempt to generate a BOLT11 invoice using the `bolt11_invoice` handler. Any errors are logged, and if the invoice creation fails, the function returns an Error.

```rust
let bolt11_invoice = match self.bolt11_invoice.receive(amount_msats, message, expiry_sec) {
   Ok(invoice) => Some(invoice),
   Err(e) => {
    log_error!(self.logger, "Failed to create invoice {}", e);
    return Err(Error::InvoiceCreationFailed);
   },
  };
```

Once we have the necessary components, we combine them into a single URI. The `LnUri::with_extras` method creates a new URI with the given on-chain address and extra parameters (BOLT11 invoice and BOLT12 offer). And finally, the function returns the URI as a string.

```rust
let extras = Extras { bolt11_invoice, bolt12_offer };
let mut uri = LnUri::with_extras(onchain_address, extras);
uri.amount = Some(Amount::from_sat(amount_sats));
uri.message = Some(message.into());

Ok(capitalize_qr_params(uri))
```

After all of that, the `receive` function successfully generates a BIP21 URI that can handle both on-chain and lightning payments with both an invoice and an offer. In the end the complete function looks like this:

```rust
pub fn receive(
        &self, amount_sats: u64, message: &str, expiry_sec: u32,
    ) -> Result<String, Error> {
        let onchain_address = self.onchain_payment.new_address()?;

        let amount_msats = amount_sats * 1_000;

        let bolt12_offer = match self.bolt12_payment.receive(amount_msats, message) {
            Ok(offer) => Some(offer),
            Err(e) => {
                log_error!(self.logger, "Failed to create offer: {}", e);
                None
            },
        };

        let bolt11_invoice = match self.bolt11_invoice.receive(amount_msats, message, expiry_sec) {
            Ok(invoice) => Some(invoice),
            Err(e) => {
                log_error!(self.logger, "Failed to create invoice {}", e);
                None
            },
        };

        let extras = Extras { bolt11_invoice, bolt12_offer };

        let mut uri = LnUri::with_extras(onchain_address, extras);
        uri.amount = Some(Amount::from_sat(amount_sats));
        uri.message = Some(message.into());

        Ok(capitalize_qr_params(uri))
    }
```

#### FORMATTING

The `capitalize_qr_params` function is designed to enhance the readability of BIP21 URIs by capitalizing the alphabetical characters in the lightning parameter. It starts by formatting the URI into a string. The function then iterates through the URI to find each occurrence of the `lightning=` part. For each occurrence, it calculates the starting and ending indices of the lightning parameter’s value, extracts this value, converts it to uppercase, and replaces the original value in the URI string. The loop continues from the end of the last found lightning parameter until no more occurrences are found. Finally, the function returns the updated URI string with all lightning parameter values capitalized for improved readability on mobile devices. In the next fix, I enhanced this function to accept both the URI and a key parameter, allowing us to pass in the key values so it could process any key-value pair (like `lno=`)!

```rust
fn capitalize_qr_params(uri: bip21::Uri<NetworkChecked, Extras>) -> String {
    let mut uri = format!("{:#}", uri);

    let mut start = 0;
    while let Some(index) = uri[start..].find("lightning=") {
        let start_index = start + index;
        let end_index = uri[start_index..].find('&').map_or(uri.len(), |i| start_index + i);
        let lightning_value = &uri[start_index + "lightning=".len()..end_index];
        let uppercase_lighting_value = lightning_value.to_uppercase();
        uri.replace_range(start_index + "lightning=".len()..end_index, &uppercase_lighting_value);

        start = end_index
    }
    uri
}
```

And of course, I’ll provide updates on any changes made to the code. Since these posts are somewhat delayed, I know that by the next post, there will be more effective ways to handle errors, rather than simply logging and silently ignoring them. Plus, the code you see is the result of multiple attempts, many of which involved dropping commits, so I can’t show you those unfortunately. It looks polished now, but I promise it took a lot of research and reviews to arrive at a decently rough draft.

### LAST WEEK UPDATES

#### SERIALIZEPARAMS UPDATE

Last week, the project used a single `lightning` key for both BOLT11 invoices and BOLT12 offers in the `SerializeParams` implementation. But, I recently discovered that BOLT12 offers should be identified by the `lno` key instead. To fix this, I made the necessary changes to the `serialize_params` function within the `SerializeParams` implementation. Previously, the implementation was:

```rust
fn serialize_params(self) -> Self::Iterator {
        let mut params = Vec::new();

        if let Some(bolt11_invoice) = &self.bolt11_invoice {
            params.push(("lightning", bolt11_invoice.to_string()));
        }
        if let Some(bolt12_offer) = &self.bolt12_offer {
            params.push(("lightning", bolt12_offer.to_string()));
        }

        params.into_iter()
    }
```

I updated it to add the `lno` key for the BOLT12 offer, resulting in the following implementation:

```rust
fn serialize_params(self) -> Self::Iterator {
    let mut params = Vec::new();

    if let Some(bolt11_invoice) = &self.bolt11_invoice {
       params.push(("lightning", bolt11_invoice.to_string()));
    }
    if let Some(bolt12_offer) = &self.bolt12_offer {
       params.push(("lno", bolt12_offer.to_string()));
    }

    params.into_iter()
}
```

This change (although simple) makes sure that BOLT12 offers are correctly identified with the `lno` key, aligning with the proper BIP21 update (WIP).

#### DESERIALIZATION UPDATE:

Last week, for deserializing extra parameters, I implemented the `DeserializeParams` trait to convert the serialized data back into the `Extras` struct. The old implementation:

```rust
fn is_param_known(&self, key: &str) -> bool {
  key == "lightning"
 }

 fn deserialize_temp(
  &mut self, key: &str, value: Param<'_>,
 ) -> Result<ParamKind, <Self::Value as DeserializationError>::Error> {
  if key == "lightning" {
   let lighting_str =
    String::try_from(value).map_err(|_| Error::UriParameterParsingFailed)?;

   for param in lighting_str.split('&') {
    if let Ok(offer) = param.parse::<Offer>() {
     self.bolt12_offer = Some(offer);
    } else if let Ok(invoice) = param.parse::<Bolt11Invoice>() {
     self.bolt11_invoice = Some(invoice);
    }
   }
   Ok(bip21::de::ParamKind::Known)
  } else {
   Ok(bip21::de::ParamKind::Unknown)
  }
 }
```

But, I had to make a change because of the new BOLT12 key. To address this, I updated the implementation to recognize both `lightning` and `lno` keys. This change ensures that the deserialization process correctly determines BOLT11 invoices and BOLT12 offers, processing each key appropriately. Here’s the update:

```rust
fn is_param_known(&self, key: &str) -> bool {
        key == "lightning" || key == "lno"
    }

    fn deserialize_temp(
        &mut self, key: &str, value: Param<'_>,
    ) -> Result<ParamKind, <Self::Value as DeserializationError>::Error> {
        match key {
            "lightning" => {
                let bolt11_value =
                    String::try_from(value).map_err(|_| Error::UriParameterParsingFailed)?;
                if let Ok(invoice) = bolt11_value.parse::<Bolt11Invoice>() {
                    self.bolt11_invoice = Some(invoice);
                    Ok(bip21::de::ParamKind::Known)
                } else {
                    Ok(bip21::de::ParamKind::Unknown)
                }
            },
            "lno" => {
                let bolt12_value =
                    String::try_from(value).map_err(|_| Error::UriParameterParsingFailed)?;
                if let Ok(offer) = bolt12_value.parse::<Offer>() {
                    self.bolt12_offer = Some(offer);
                    Ok(bip21::de::ParamKind::Known)
                } else {
                    Ok(bip21::de::ParamKind::Unknown)
                }
            },
            _ => Ok(bip21::de::ParamKind::Unknown),
        }
    }
```

### WRAP-UP

In this post, we went through the implementation of the `receive` function, which generates a BIP21 URI capable of handling both on-chain and lightning payments using BOLT11 invoices and BOLT12 offers. We dove into the step-by-step process of generating the necessary components and combining them into a single URI, ensuring decent (better to come) error handling and improved readability for mobile devices.

In the next post, I’ll write about the `send` function and how I implemented it. I’ll also cover where these methods are called within LDK Node, and how I exposed the APIs in LDK Node’s bindings to facilitate easy integration with Python and Kotlin.

As always, thanks for joining me and if you found this article insightful or have any questions or critiques, I’d love to hear from you. Feel free to reach out to me on Twitter or LinkedIn. If you want to review my PR, check it out here.
