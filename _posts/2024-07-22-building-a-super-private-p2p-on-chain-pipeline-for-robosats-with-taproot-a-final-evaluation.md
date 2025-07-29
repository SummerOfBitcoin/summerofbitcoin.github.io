---
layout: post
title: "Building a Super Private P2P On-Chain Pipeline for RoboSats with Taproot: A Final Evaluation"
date: 2024-07-22
author: "Aarav Mehta"
categories: [Robosats, Stories]
image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1724934512043/fcf0c2fb-b60a-4f31-ae13-359e04f59124.png"
---
## **Abstract**

Hello there! I’m Aarav Mehta, studying Maths and Computing at IIT BHU. This year, I got the chance to participate in Summer of Bitcoin ‘24 in the Robosats organization, for a very exciting project of building Super private P2P onchain pipeline for RoboSats using Taproot Transaction, along with a very talented partner, [f321x](https://github.com/f321x).

This blog is for the stuff I did after my midterm evaluation, and what I am submitting for my final evaluation for Summer of Bitcoin. You can see my pre mid evaluation blog here: [https://aaravmehta.hashnode.dev/halfway-through-summer-of-bitcoin-2024](https://aaravmehta.hashnode.dev/halfway-through-summer-of-bitcoin-2024)

## **My Project**

RoboSats is a simple FOSS tool to privately exchange bitcoin for national currencies. Robosats simplifies the peer-to-peer user experience and uses lightning hold invoices to minimize custody and trust requirements. Anyone can be a robosats user and any lightning node can become a robosats provider, no barriers no costs.

My basic task was to use taproot contracts to build a super-private p2p onchain pipeline that will look just like a basic P2TR if the trade goes well (no disputes), using BDK library.

Here is the link for the project: [https://github.com/RoboSats/taptrade-core](https://github.com/RoboSats/taptrade-core)

## **Work done**

The major work done by me was trying to build the major workflow of the P2TR transaction, since the communication and the data pipe lining part was over. As I had mentioned in the previous blog, before the mid evaluation, I had completed making the descriptor function, and tried to fund the P2TR transaction. After mid evaluation I worked on completing the function, and added some docs to the existing functions.

Essentially, there are 4 steps to building a P2TR transaction:

1. Creating the descriptor
    
2. Funding the transaction
    
3. Cashing out the transaction, either using musig or alternate spending paths.
    
    1. For this step, firstly the required parties give their signature
        
    2. Then, the signature are aggregated with the fees
        
4. Combining and broadcasting the transaction.
    

I had already discussed the first 2 steps in my previous blog. Here, I will tell you how to do the other 2 steps.

### Cashing Out

There are multiple ways to cash out the P2TR transaction. This was my original approach:

```rust
// Parse the xprv key
let xprv = ExtendedPrivKey::from_str(xprv_str)?;

// Derive the private key (you can adjust the derivation path as needed)
let derivation_path = DerivationPath::from(vec![ChildNumber::from(0)]);
let derived_priv_key = xprv.derive_priv(&secp, &derivation_path)?;

// Convert the derived key to a PrivateKey
let private_key = PrivateKey::new(derived_priv_key.private_key, Network::Bitcoin);


let signer = SignerWrapper::new(
	private_key,
	SignerContext::Tap {
		is_internal_key: false,
	},
);

wallet.add_signer(KeychainKind::External, SignerOrdering(0), Arc::new(signer));

// Step 3: Sign the transaction
let finalized = wallet.sign(&mut psbt, SignOptions::default());
```

This way the user can easily cash out the transaction by signing the particular alternate path.

But, finally the signing out of the transaction was done like this:  

```rust
// obtain the message to sign (taproot key spend signature hash)
let msg = {
	let mut sig_hash_cache = SighashCache::new(keyspend_psbt.unsigned_tx.clone());

	// it should only contain one utxo, the escrow locking UTXO
	let utxo = keyspend_psbt
		.iter_funding_utxos()
		.next()
		.ok_or(anyhow!("No UTXO found in payout psbt"))??
		.clone();

	let sighash_type = keyspend_psbt.inputs[0].taproot_hash_ty()?;
   	// get the msg (sighash) to sign with the musig key
	let binding = sig_hash_cache
		.taproot_key_spend_signature_hash(0, &Prevouts::All(&[utxo]), sighash_type)
		.context("Failed to create keyspend sighash")?;
	binding.to_raw_hash()
};
```

### Combining and Broadcasting the transaction

Again, there are multiple ways to combine and broadcast the transaction. I was originally planning to broadcast on Esplora, like most of the other bitcoin repositories, but later on, we moved the entire thing to regtest node for better testing purposes.

```rust
for psbt in signed_psbts {
	let psbt = PartiallySignedTransaction::from_str(psbt)?;
    base_psbt.combine(psbt)?;
}

let secp = Secp256k1::new();
let psbt = base_psbt.finalize(&secp).unwrap();
let finalized_tx = psbt.extract_tx();
dbg!(finalized_tx.txid());

let blockchain = EsploraBlockchain::new("https://blockstream.info/testnet/api", 20);
dbg!(blockchain.broadcast(&finalized_tx));
```

So, instead of having the last 2 lines, where the transaction is deployed on Esplora, we can finalise and broadcast the transaction in our Regtest node like this:

```rust
maker_psbt.combine(taker_psbt)?;
debug!("Combined escrow psbt: {:#?}", maker_psbt);

let wallet = self.wallet.lock().await;
match wallet.finalize_psbt(&mut maker_psbt, SignOptions::default()) {
	Ok(true) => {
		let tx = maker_psbt.extract_tx();
		self.backend.broadcast(&tx)?;
		info!("Escrow transaction broadcasted: {}", tx.txid());
		Ok(())
	}
	Ok(false) => Err(anyhow!("Failed to finalize escrow psbt")),
	Err(e) => Err(anyhow!("Error finalizing escrow psbt: {}", e)),
}
```

### Finishing the project

For finishing the project, I added a description of how to use this repository in the README and some rustdocs and doc tests in the already created files and functions.

## Learning

* Throughout this journey, I learned a lot about the various things happening in bitcoin, a lot of new terminologies, technologies and interesting ideas and proposals related to bitcoin. I also came to know about a lot of people in bitcoin because of this (I finally joined nostr: npub1afrdqq699c2ygl3acndq76gzxdzyvdqlflz9ge2p509wpksfqc6s27rdr5 🙂)
    
* I was able to look into bitcoin transactions: types of transactions, how to create them, how to analyze and verify them.
    
* Another major learning for me was getting fairly proficient in Rust programming and programming in Bitcoin-architecture applications.
    

## Special Thanks

It was an awesome experience for me to work on this project. I learned a lot of new things, and Felix's ([@f321x](https://x.com/f321x_)) knowledge was instrumental in completing the project. Also, thanks to my mentors [@recksato](https://x.com/recksato) and [@koala\_sat](https://x.com/koala_sat), for giving direction for what they want in the project.

I would also like to thank Adi Shankara and Summer of Bitcoin for this amazing learning experience. I was able to develop great skills and valuable experience. I look forward to contributing to the Bitcoin space.

# **Conclusion**

Thank you for reading, hope you enjoyed it. I'll continue to update my progress via the series of blogs ;)

Follow me on [Twitter](https://x.com/aaravchessfan) | [Linkedln](https://www.linkedin.com/in/aarav-mehta-445959250/) for more blockchain-development tips and posts.

That's all for today!