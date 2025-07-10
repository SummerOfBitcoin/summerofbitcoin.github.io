---
layout: post
title: Improving coin selection in BDK
author: César Alvarez Vallero
date: "2022-08-19 13:28:27 +0000"
categories:
  - "BDK"
  - "Coin Selection"
---

As a project designed to be used in wallet development, one of the main things that BDK provides is the coin selection module. The purpose of the module is to select the group of utxos to use as inputs for the transaction. When you coin select you must consider cost, size and traceability.

### What are those costs?

Principally fees. Determined by the satisfactory size required of each of the inputs as well as the change outputs generated. Change outputs are not part of the inputs, but they must be considered during coin selection because they affect the feerate of the transaction and will be used in future transactions as inputs.For example, if you create change outputs every time, you'll probably end up with a lot of very small UTXOs. The smaller the UTXO, the greater the proportion of fees spent to use that UTXO, depending on the fee rate.

### What do we mean by "size" considerations?

Here we are not referring to the size in MB of the transaction, as that is addressed by the associated fees. Here, "size" is the number of new UTXOs created by each transaction. It has a direct impact on the size of the UTXO set maintained by each node.

### What is this traceability thing?

Certain companies sell services whose purpose is to link address with their owners, harming the fungibility of some bitcoins and attacking the privacy of the users. There are some things that coin selection can do to make privacy leaking harder. For example, not creating change outputs, avoiding mixing UTXOs belonging to different owned addresses in the same transaction, or the total expenditure of the related utxos.

Besides the algorithm you use to coin select, which can impact some of the things described above, other code changes also have implications. The following sections will describe some of those changes and why they have been done or could be added.

## Waste

One of the changes projected for the `coin_selection` module is the addition of the `Waste` metric, and its use in optimizing coin selection in relation to the fee costs.

Waste is a metric introduced by the BnB algorithm as part of its bounding procedure. Later, it was included as a high-level function to use in comparison of different coin selection algorithms in Bitcoin Core.

### How it works?

We can describe waste as the sum of two values: creation cost and timing cost.

Timing cost is the cost associated with the current fee rate and some long-term fee rate used as a threshold to consolidate UTXOs. It can be negative if the current fee rate is cheaper than the long-term fee rate or zero if they are equal.

Creation cost is the cost associated with the surplus of coins besides the transaction amount and transaction fees. It can happen in the form of a change output or excessive fees paid to the miner. Change cost derives from the cost of adding the extra output to the transaction and spending in the future. Excess happens when there is no change, and the surplus of coins is spent as part of the fees to the miner.

The creation cost can be zero if there is a perfect match as a result of the  
coin selection algorithm. So, waste can be zero or negative if the creation cost is zero and the timing cost is less than or equal to zero.

You can read its technical aspects in its [PR](https://github.com/bitcoindevkit/bdk/pull/558?ref=blog.summerofbitcoin.org). Comments and suggestions are  
welcome!

### What has been done

Waste is closely related to the creation of change or the drop of it as fees.  
Formerly, whether your selection would produce change or not was decided inside the `create_tx` function. From the perspective of the Waste metric, that was problematic. How to score coin selection based on `Waste` if you don't know yet that it will create change or not?

The problem had been pointed out before, in [this issue](https://github.com/bitcoindevkit/bdk/issues/147?ref=blog.summerofbitcoin.org).

A [PR](https://github.com/bitcoindevkit/bdk/pull/630?ref=blog.summerofbitcoin.org) merged in the last release moved change creation to the `coin_selection` module. It introduced several changes:

* the enum `Excess`.
* the function `decide_change`.
* a new field in `CoinSelectionResult` to hold the `Excess` produced while coin  
  selecting.

We hope to have chosen meaningful names for all these new additions, but lets  
explain them in depth.

Formerly, when you needed to create change inside `create_tx`, you must get the  
weight of the change output, compute its fees and, jointly with the overall fee amount and the outgoing amount, subtract them from the remaining amount of the selected utxos, then decide whether the amount of that output should be considered dust, and throw the remaining amount to fees in that case. Otherwise add an extra output to the output list and sum their fees to the fee amount.

Also, there was the case when you wanted to sweep all the funds associated with an address, but the amount created a dust output. In that situation, the dust value of the output and the amount available after deducing the fees were necessary to report an informative error to the user.

In general, the idea was to compute all those values inside `coin_selection` but keep the decision logic where it was meaningful, that is, inside `create_tx`.

Those considerations ended up with an enum, `Excess`, with two struct variants that differentiated the cases mentioned above, which carry all the needed information to act in each one of those cases.

The function `decide_change` was created to build `Excess`. This function requires the remaining amount after coin selection, the script that will be used to create the output and the fee rate aimed by the user. To pass this new value to `Wallet::create_tx` and make decisions based on it, the field `excess` was added to the `CoinSelectionResult`, and the `coin_select` methods of each algorithm were adapted to compute this value, using `decide_change` after performing the coin selection.

### Work in progress

It is necessary to integrate the `Waste::calculate` method with the `CoinSelectionAlgorithm` implementations and the `decide_change` function.

The first step in that direction would be the removal of the Database generic parameter from the `CoinSelectionAlgorithm` trait. There isn't a clear way to make it, as you may guess by this [issue](https://github.com/bitcoindevkit/bdk/issues/281?ref=blog.summerofbitcoin.org). The only algorithm currently using the database features is `OldestFirstCoinSelection`.

There is a proposal to fix this problem by removing the need of a database trait altogether, so, in the meanwhile, we could move the generic from the trait to the `OldestFirstCoinSelection`, to avoid doing work that will probably be disposed in the future.

Another step in that way is a proposal to add a `CoinSelectionAlgorithm::get_selection` wrapper to the coin selection module, which will join together preprocessing and validation of the utxos, coin selection, the decision to create change and the calculus of waste in the same function. The idea is to create a real pipeline to build a `CoinSelectionResult`.

In addition, the function will allow the separation of the algorithms `BranchAndBound` and `SingleRandomDraw` from each other, which were put together only by the dependence of the former on the second one as a fallback method.  
That dependence will not be broken, but the possibility to use `SingleRandomDraw` through BDK for their purpose will be left open, expanding the flexibility of the library.

As a bonus, this function will deflate some parts of the code from unnecessary information, avoid code duplication (and all the things associated with it) and provide a simple interface to integrate your custom algorithms with all the other functionalities of the BDK library, enhancing them through the new change primitives and the computation of Waste.

Watch out for the PR!

## Further Improvements

Besides the `Waste` metric, there are other changes that could improve the  
current state of the coin selection module in BDK, which will impact the  
privacy and the flexibility provided by it.

### Privacy

In Bitcoin Core, the term `Output Group` is associated with a structure that joins all the UTXOs belonging to a certain ScriptPubKey, up to a specified threshold. The idea behind this is to reduce the address footprint in the blockchain, reducing traceability and improving privacy.  
In BDK, OutputGroups are a mere way to aggregate metadata to UTXOs. But this structure can be improved to something like what there is in Bitcoin, by transforming the weighted utxos in a vector of them and adding a new field or parameter to control the amount stored in the vector.

### Flexibility

A further tweak in the UTXO structure could be the transition to traits, which  
defines the minimal properties accepted by the algorithms to select the  
underlying UTXOs.

The hope is that anyone can define new algorithms consuming any form of UTXO  
wrapper that you can imagine, as long as they follow the behavior specified by  
those primitive traits.

Also, there is a major architectural change proposal called `bdk_core` that will refactor a lot of sections of BDK to improve its modularity and flexibility. If you want to know more, you can read the  
[blog post](https://bitcoindevkit.org/blog/bdk-core-pt1/?ref=blog.summerofbitcoin.org) about it or dig  
directly into its [prototype](https://github.com/LLFourn/bdk_core_staging?ref=blog.summerofbitcoin.org).

## Conclusion

A lot of work is coming to the coin selection module of BDK.  
Adding the Waste metric would be a great step in the improvement of the coin selection features of the kit, and we hope to find new ways to measure the selection capabilities. We are open to new ideas!

The new changes range from refactorings to enhancements. It's not hard to find  
something to do in the project, as long as you spend some time figuring out how the thing works. Hopefully, these new changes will make this task easier. And we are ready to help if you need to.

If you would like to improve something, request a new feature or discuss how  
would you use BDK in your personal project, join us on  
[Discord](https://discord.gg/dstn4dQ?ref=blog.summerofbitcoin.org).

## References

### About coin selection considerations

* Jameson Lopp. "The Challenges of Optimizing Unspent Output Selection"  
  *Cypherpunk Cogitations*.  
  [https://blog.lopp.net/the-challenges-of-optimizing-unspent-output-selection/](https://blog.lopp.net/the-challenges-of-optimizing-unspent-output-selection/?ref=blog.summerofbitcoin.org)

### About Waste metric

* Murch. "What is the Waste Metric?" *Murch ado about nothing*.  
  [https://murch.one/posts/waste-metric/](https://murch.one/posts/waste-metric/?ref=blog.summerofbitcoin.org)
* Andrew Chow. "wallet: Decide which coin selection solution to use based on  
  waste metric" *Bitcoin Core*. [https://github.com/bitcoin/bitcoin/pull/22009](https://github.com/bitcoin/bitcoin/pull/22009?ref=blog.summerofbitcoin.org)
* Bitcoin Core PR Review Club. "Decide which coin selection solution to use  
  based on waste metric". *Bitcoin Core PR Review Club*.  
  [https://bitcoincore.reviews/22009](https://bitcoincore.reviews/22009?ref=blog.summerofbitcoin.org)

### About improving privacy in coin selection

* Josi Bake. "wallet: avoid mixing different OutputTypes during coin selection"  
  *Bitcoin Core*. [https://github.com/bitcoin/bitcoin/pull/24584](https://github.com/bitcoin/bitcoin/pull/24584?ref=blog.summerofbitcoin.org)
* Bitcoin Core PR Review Club. "Increase OUTPUT\_GROUP\_MAX\_ENTRIES to 100"  
  *Bitcoin Core PR Review Club*. [https://bitcoincore.reviews/18418](https://bitcoincore.reviews/18418?ref=blog.summerofbitcoin.org)
* Bitcoin Core PR Review Club. "Avoid mixing different `OutputTypes` during  
  coin selection" *Bitcoin Core PR Review Club*.  
  [https://bitcoincore.reviews/24584](https://bitcoincore.reviews/24584?ref=blog.summerofbitcoin.org)

### About `bdk_core`

* Lloyd Fournier. "bdk\_core: a new architecture for the Bitcoin Dev Kit".  
  *bitcoindevkit blog*. [https://bitcoindevkit.org/blog/bdk-core-pt1/](https://bitcoindevkit.org/blog/bdk-core-pt1/?ref=blog.summerofbitcoin.org)
