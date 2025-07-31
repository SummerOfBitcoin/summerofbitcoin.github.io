---
layout: post
title: Implementing Pubkey and Duration-Based Token Locking for Cashu in Zeus Wallet
date: 2025-07-08
author: Ajay Sehwal
categories: [Zeus, Cashu]
image: ../assets/images/blog_content/2025-07-08-implementing-pubkey-and-duration-based-token-locking-for-cashu-in-zeus-wallet_f98de17e.png
---

## Step 1: You Send Bitcoin to the Mint

Your wallet sends a **Lightning payment** (or on-chain Bitcoin, depending on
the mint) to the Cashu **mint** — the server that issues and verifies tokens.

Think of the mint like a digital shop — you give it Bitcoin, and you’ll get
ecash tokens in return.

**Step 2: The Mint Creates and Signs Your Token — Without Seeing It.**

Your wallet creates a **blinded request** — a clever way of asking for tokens
without showing the mint what it’s signing. The mint **signs** this request
and sends it back to you.

> Think of it like putting your request inside a locked box — the mint can
> sign the outside, but it can’t see what’s inside.

## Step 3: You Now Have a Private Cashu Token

Your wallet **unblinds** the signed token, and now it’s ready to use.

You can:

  * 💸 Send it to someone
  * ☕ Spend it
  * 🔁 Redeem it for Bitcoin  
— and the mint **can’t track** who you are or what you’re doing with it

# **Why Do We Need Token Locking?**

Cashu tokens are great — they’re fast, private, and easy to use, just like
digital cash.  
But here’s the thing:

> Anyone who holds a token can spend it.

This is by design — like handing someone a ₹100 note. But sometimes, you want
**more control** over who can use that note and when.

Let’s say you generate a Cashu token and send it to someone?

  * What if someone else gets access to it (accidentally or maliciously)?
  * What if you want it to be used only by one specific person?

**That’s why we introduced a powerful new feature in Cashu: token locking.**  
Using this, you can:

  * 🔐 Lock a token to a specific public key — so only the intended user can spend it using their wallet.
  * ⏱️ Set a duration (expiry time) — so the token can only be used within a limited time window.

This means you now have full control over **who can use the token** and **for
how long** it stays valid.

# How I Built the Token Locking Feature in Zeus Wallet

Implementing token locking in Zeus Wallet was a challenging yet rewarding
experience. The goal was to allow users to **lock tokens to a specific pubkey
with an optional unlock time** , enhancing security and usability for future
recovery or scripting purposes.

I broke the feature down into **three main steps** :

## 🔧 Step 1: Update the `mintToken` Function

To begin, I extended the existing `mintToken` function to accept additional
fields — a `pubkey` and an optional `lockTime`.

    
    
    pubkey?: string;  
    lockTime?: number;

These are passed in from the UI where the user can input a recovery key and
choose an unlock duration (in seconds or as a Unix timestamp). If present, the
code constructs a `p2pk` object like this:

    
    
    const p2pk = pubkey  
      ? {  
            pubkey,  
            ...(lockTime && { locktime: lockTime })  
        }  
      : undefined;

This The `p2pk` object encapsulates the lock conditions and gets passed to the
minting logic, so the minted token is bound by these constraints.

    
    
        getSpendingProofsWithPreciseCounter = async (  
            wallet: CashuWallet,  
            amountToPay: number,  
            proofs: Proof[],  
            currentCounter: number,  
            swap?: boolean,  
            p2pk?: { pubkey: string; locktime?: number; refundKeys?: Array<string> }  
        ) => {  
            try {  
                const { keep: proofsToKeep, send: proofsToSend } = swap  
                    ? await wallet.swap(amountToPay, proofs, {  
                          keysetId: wallet.keysetId,  
                          counter: p2pk ? undefined : currentCounter,  
                          includeFees: true,  
                          p2pk  
                      })  
                    : await wallet.send(amountToPay, proofs, {  
                          keysetId: wallet.keysetId,  
                          counter: p2pk ? undefined : currentCounter,  
                          includeFees: true,  
                          p2pk  
                      });  
      
                const existingProofsIds = proofs.map((p) => p.secret);  
                const newKeepProofsCount = proofsToKeep.filter(  
                    (p) => !existingProofsIds.includes(p.secret)  
                ).length;  
                const newSendProofsCount = proofsToSend.filter(  
                    (p) => !existingProofsIds.includes(p.secret)  
                ).length;  
      
                console.log(newKeepProofsCount, 'NEW PROOFS KEEP COUNT');  
                console.log(newSendProofsCount, 'NEW PROOFS SEND COUNT');  
      
                const newCounterValue =  
                    currentCounter + newKeepProofsCount + newSendProofsCount;  
      
                return {  
                    proofsToSend,  
                    proofsToKeep,  
                    newCounterValue  
                };  
            } catch (err) {  
                console.error('Error in getSpendingProofsWithPreciseCounter:', err);  
                throw err;  
            }  
        };

## 🔁 Step 2: Pass Lock Info to `getSpendingProofsWithPreciseCounter`

During the minting process, I reused our internal function:This function
either sends or swaps the required amount using existing proofs, but now I
modified it to accept and pass down the p2pk object.

Inside this function, the token-lock logic is injected:

    
    
    await wallet.send(amountToPay, proofs, {  
        keysetId: wallet.keysetId,  
        counter: p2pk ? undefined : currentCounter,  
        includeFees: true,  
        p2pk  
    })

This is where the magic happens. By passing the `p2pk` object into the
`wallet.send` call, the newly minted token becomes cryptographically locked to
the provided pubkey and (if given) unlockable only after the `locktime`.

## 🕒 Step 3: Duration Handling and UI

On the UI side, I allowed users to:

  * Enter a **recovery pubkey**
  * Set a **duration** or **unlock time**

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OEZhwek57734qZG-qjDyzw.png"/>
</figure>

The duration is converted to a Unix timestamp before passing it as `lockTime`
to the backend logic. This ensures tokens can be locked for, say, 1 hour, 1
day, or even until a specific date, enhancing flexibility.

# ✅ Result

The final minted token now includes:

  * A **p2pk script** with a `pubkey` and optional `locktime`
  * Which makes it **non-spendable** until that time or without the correct pubkey

# Now time to Claiming or Spending the Locked Token :

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dmbNkD5D-u45r44go_cf0g.png"/>
</figure>

# How Zeus Verifies and Claims a Locked Token :

    
    
    async function claimToken(encodedToken: string, decoded: CashuToken) {  
        const mintUrl = decoded.token[0].mint;  
        const wallet = this.cashuWallets[mintUrl];  
        const walletPubkey = wallet.pubkey;  
      
        const isLocked = CashuUtils.isTokenP2PKLocked(decoded);  
        let isLockedToWallet = false;  
      
        if (isLocked) {  
            const firstLockedPubkey = CashuUtils.getP2PKPubkeySecret(decoded.proofs[0].secret);  
      
            const allProofsLockedToSamePubkey = decoded.proofs.every((proof) => {  
                const proofPubkey = CashuUtils.getP2PKPubkeySecret(proof.secret);  
                return proofPubkey === firstLockedPubkey;  
            });  
      
            if (!allProofsLockedToSamePubkey) {  
                this.loading = false;  
                return {  
                    success: false,  
                    errorMessage: localeString('stores.CashuStore.claimError.inconsistentLocking')  
                };  
            }  
      
            isLockedToWallet = firstLockedPubkey === walletPubkey;  
            if (!isLockedToWallet) {  
                this.loading = false;  
                return {  
                    success: false,  
                    errorMessage: localeString('stores.CashuStore.claimError.lockedToWallet')  
                };  
            }  
        }  
      
        // Prepare key for claiming  
        const counter = wallet.counter + decoded.proofs.length;  
        const currentSeed = this.getSeed();  
        const currentPrivkey = Base64Utils.base64ToHex(  
            Base64Utils.bytesToBase64(currentSeed.slice(0, 32))  
        );  
      
        const currentDerivedPubkey = '02' + bytesToHex(schnorr.getPublicKey(currentPrivkey));  
        if (currentDerivedPubkey !== walletPubkey) {  
            this.loading = false;  
            return {  
                success: false,  
                errorMessage: localeString('stores.CashuStore.claimError.keyMismatch')  
            };  
        }  
      
        // Receive token  
        const newProofs = await wallet.receive(encodedToken, {  
            keysetId: wallet.keysetId,  
            privkey: currentPrivkey,  
            counter  
        });  
      
        const amtSat = CashuUtils.sumProofsValue(newProofs);  
        const newCounter = counter + newProofs.length;  
      
        const totalBalanceSats = new BigNumber(this.totalBalanceSats || 0)  
            .plus(amtSat || 0)  
            .toNumber();  
      
        const balanceSats = new BigNumber(wallet.balanceSats || 0)  
            .plus(amtSat || 0)  
            .toNumber();  
      
        // Update proofs, balances, and counters  
        wallet.proofs.push(...newProofs);  
        await this.setMintProofs(mintUrl, wallet.proofs);  
        await this.setMintCounter(mintUrl, newCounter);  
        await this.setMintBalance(mintUrl, balanceSats);  
        await this.setTotalBalance(totalBalanceSats);  
      
        // Record received token  
        this.receivedTokens?.push(  
            new CashuToken({  
                ...decoded,  
                received: true,  
                encodedToken,  
                received_at: Date.now() / 1000  
            })  
        );  
        await Storage.setItem(`${this.getLndDir()}-cashu-received-tokens`, this.receivedTokens);  
      
        this.checkAndSweepMints(mintUrl);  
        this.loading = false;  
      
        return { success: true, errorMessage: '' };  
    }  
    

## 1\. Is This Token Locked?

We first check if the token is **P2PK (pubkey) locked** using:

    
    
    CashuUtils.isTokenP2PKLocked(decodedToken)

If yes, we know this token can **only be claimed by the wallet that owns the
matching pubkey**.

## 2\. Are All Token Proofs Locked to the Same Pubkey?

Each token can include multiple “proofs” (like coins in a bag). We verify that
**all of them are locked to the same pubkey** :

    
    
    const allProofsLockedToSamePubkey = decoded.proofs.every((proof) => {  
      return CashuUtils.getP2PKPubkeySecret(proof.secret) === firstLockedPubkey;  
    });

This prevents people from trying to mix in random or fake proofs.

## 3\. Does This Wallet Own That Pubkey?

Next, we check if **this wallet actually owns the private key** that can
unlock the token.  
We do that by **deriving the wallet’s current pubkey** from its seed:

If it doesn’t match the pubkey the token is locked to — sorry, this token
isn’t for you.

## 4\. Check Lock Duration (If Any)

If a **lock duration** was set when the token was created, we make sure
**enough time has passed** before allowing the claim.

This prevents early claiming and enforces a “wait until…” rule ⏰

## 5\. Claim It Into the Wallet

Once everything checks out — the pubkey matches and time is valid — we
**finally redeem the token** into the Zeus Wallet:

    
    
    await wallet.receive(encodedToken, {  
      keysetId,  
      privkey: currentPrivkey,  
      counter,  
    });

The token is now stored securely in your wallet and ready to be spent, sent,
or swapped later.

Check out the demo video below to see how you can Lock Cashu Token to Pubkey
with duration in Zeus wallet :

<https://youtube.com/shorts/Y_cluvuhTqI?feature=share>

Conclusion:

With the introduction of **token locking in Zeus Wallet** , we’re stepping
into a more secure and permissioned eCash experience. Locking tokens to a
specific **pubkey** (and optionally to a **duration**) ensures that only the
rightful wallet owner can claim or spend the token — even if someone else gets
access to the token string.

This is a powerful step toward **secure, flexible, and programmable digital
cash**. Whether you’re building apps on top of Cashu or just want tighter
control over your tokens, this feature opens up exciting new use cases.

