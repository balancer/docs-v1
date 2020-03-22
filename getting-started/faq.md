# FAQ

## Basics

### What is Balancer Protocol?

Balancer is a protocol for multi-token [automated market-making](../protocol/background.md). It enables portfolio owners to create Balancer Pools and traders to trade against these pools. Balancer Pools contain 2 or more tokens each with arbitrary weights of the total pool value. The pools provide the Balancer Protocol with liquidity, and charge traders a trading fee in return. Pools can be considered automated market-makers since anyone can trade between any two tokens present in any pool.

### What can Balancer Protocol be useful for?

There are two categories of users who can benefit from Balancer Protocol: liquidity providers ‒ who own Balancer Pools or participate in shared pools ‒ and traders ‒ who use pools to trade.

Liquidity providers can be really any holder of 2 or more ERC20 tokens like:

* Portfolio managers who want to have controlled exposure to different assets without having to deal with complicated and/or expensive rebalancing procedures 
* Investors who have ERC20 tokens sitting idly in a wallet and would like to put them to work so they can effortlessly earn fees 

Traders can benefit from the diverse set of pools, each subject to different slippage, fees and spot prices between any pair of their tokens. They can be:

* Conventional traders seeking to exchange any token for another with the best return possible 
* Arbitrageurs seeking profit by levelling market inefficiencies between Balancer pools and other DEXs or CEXs 
* Ethereum Smart Contracts seeking liquidity for a variety of other use cases such as liquidating positions in other protocols, trading on behalf of users of other platforms, etc. 

### Is Balancer Protocol fully permission-less?

Yes. Balancer Pools cannot be censored or whitelisted. Traders cannot be censored or whitelisted. Balancer Labs does not have the power to stop or edit the smart contracts in any way after they’ve been deployed. There is no upgradeability, admin functionalities, or backdoors in the contracts.

### Is there a roadmap?

We are working on putting together a more detailed roadmap. The bronze release is planned to go live in February 2020. Silver is currently in a design phase and will likely release late 2020.

### Does Balancer Protocol charge any fees at the protocol layer?

Not in the bronze release, although the code shows how an `EXIT_FEE` may be introduced in future versions.

### Is there a Balancer Protocol token?

Not initially. There might be some experimentation towards protocol tokens and liquidity mining in future versions. A token will never be required to trade or interact with the protocol. Any protocol upgrade/migration in that regard would be discussed with the community of users of the protocol well in advance.

## Balancer Pools

### What is a Balancer Pool?

The fundamental building blocks of Balancer Protocol are Balancer Pools. Pools are smart contracts integrated with Balancer Protocol that hold value in 2 or more ERC20 tokens. They can be seen as a market-making portfolio, since each token comprising the pool has a different value weight and can be traded against any other of its tokens. An example of pool would be one with 50% WETH, 25% MKR and 25% DAI.

Two main properties make Balancer Pools special:

1. They ensure that, even as the relative unit prices of its tokens vary, they are continuously rebalanced to maintain the desired value distribution across tokens constant. 
2. Each trade that takes place in a Balancer Pool generates a fee to the pool owner. The fee is a percentage of the amount traded, and is customizable by the pool owner. 

### Are there constraints for setting up a Balancer Pool?

Only a few. Balancer Protocol limits pools in the following ways:

* Maximum number of tokens: there can only be at most 8 different tokens in the pool.
* Swap fee: the fee can be chosen to be any number between 0.0001% and 10% 
* ERC20 compliance: pool tokens have to be ERC20 compliant. Bronze does not support ERC20 tokens that do not return `bools` for `transfer` and `transferFrom`. Future releases will allow non-standard ERC20's.
* There are a few additional ratio and balance constraints that can be found at [Limitations](../protocol/limitations.md)

### How are Balancer Pools continuously rebalanced?

This is probably the main innovation brought by Balancer Protocol. Pools are efficiently rebalanced thanks to a multi-dimensional invariant function used to continuously define swap prices between any two tokens in any pool.

Imagine a portfolio that contains a proportion of token A and its price increases. A conventional rebalancing process would sell this token and consequently pay a fee for that trade. Balancer Protocol, instead, lets rational market actors actively buy token A from the pool. So instead of selling token A and paying a fee for that, the Balancer Pool let other actors buy token A and charge a fee for that.

In summary, instead of paying fees to rebalance, Balancer Pools earn fees to let others rebalance them. For further technical details, please refer to our [white paper](https://balancer.finance/whitepaper.html).

### How do Balancer Pools charge fees and how much are they?

Balancer pools charge a percentage of the input amount traded for each trade. The fee goes entirely to the Balancer Pool liquidity providers.

### What types of Balancer pools are there?

Balancer Pools can be controlled or finalized. See more at [Core Concepts](../protocol/concepts.md)

## Using Balancer Protocol

### How can I use Balancer as a trader?

There are a few ways a trader can interact with Balancer Protocol:

* Through our exchange front-end \(available shortly\)
* Directly through our smart Contracts on Ethereum mainnet

### How can I use Balancer as a liquidity provider or portfolio manager?

There are three main ways a liquidity provider or portfolio manager can interact with Balancer Protocol:

* Through our pool manager front-end \(available shortly\)
* Directly through our smart Contracts on Ethereum mainnet

Please [contact us](mailto:contact@balancer.finace) if you have a portfolio worth more than 100,000 USD and need special assistance.

### Can Balancer be used as an Index Fund?

Balancer Protocol is ideal for Index Funds as it takes away all the hassle that managers have regarding rebalancing and on top of that generates fees from trades it enables.

## Miscellaneous

### Does using Balancer Protocol generate taxable events?

We cannot provide tax or accounting advice. Tax regulations are specific to jurisdiction where you or your company reside. For any legal or tax matters we recommend consulting your own attorney.

### Are there risks in using Balancer Protocol?

Balancer Protocol smart contracts have been designed having security as its top priority. We are having our code be audited by best in class professionals. However, we cannot guarantee that bugs won’t be found in the future.

