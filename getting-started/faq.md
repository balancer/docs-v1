# FAQ

## FAQ

### What is Balancer Protocol?

Balancer is a protocol for multi-token automated market-making. It enables portfolio owners to create Balancer Pools and traders to trade against these pools. Balancer Pools contain 2 or more tokens each with arbitrary shares of the total pool value. The pools provide the Balancer Protocol with liquidity, and charge traders a trading fee in return. Pools can be considered automated market-makers since anyone can trade between any two tokens present in any pool.

See [Balancer Pools](http://a) for more details.

### What can Balancer Protocol be useful for?

There are two categories of users who can benefit from Balancer Protocol: liquidity providers ‒ who own Balancer Pools or participate in shared pools ‒ and traders ‒ who use pools to trade.

Liquidity providers can be really any holder of 2 or more ERC20 tokens like:

* Portfolio managers who want to have controlled exposure to different assets without having to deal with complicated and/or expensive rebalancing procedures 
* Investors who have ERC20 tokens sitting idly in a wallet and would like to put them to work so they can effortlessly earn fees 

Traders can benefit from the diverse set of pools, each subject to different slippage, fees and spot prices between any pair of their tokens. They can be:

* Conventional traders seeking to exchange any token for another with the best return possible 
* Arbitrageurs seeking profit by levelling market inefficiencies between DEXs and even CEXs 
* Ethereum Smart Contracts seeking liquidity for a variety of other use cases such as liquidating positions in other protocols, trading on behalf of users of other platforms, etc. 

### Is there a Balancer Protocol token?

Not initially. There might be some experimentation towards protocol tokens in future versions. Any protocol upgrade/migration in that regard would be discussed with the community of users of the protocol well in advance - multiple months, not weeks.

### Is Balancer Protocol fully permissionless?

Yes. Balancer Pools cannot be censored or whitelisted. Traders cannot be censored or whitelisted. Balancer Labs does not have the power to stop or edit the smart contracts in any way after they’ve been deployed. Please read the [result of our code audits](http://a) for more details.

### Is there a roadmap?

Yes, Balancer Protocol has already 2 future versions planned. The current one is called Bronze and is already live and functional on [Ethereum mainnet](http://a). Silver and Gold are planned for launch in 2020. See more details on [our roadmap page](http://a).

### Does Balancer Protocol charge any fees at the protocol layer?

No protocol fees are charged at all if users keep their holdings in the form of Balancer wrapped tokens, or bTokens \(for details see “[What is a Balancer Token Wrapper?](http://a)”\).

Whenever a bToken is redeemed for its underlying token, say bDAI for DAI, a very small exit fee of 0.1% is charged. This fee goes to Balancer Labs OÜ, the company that created Balancer Protocol, and is used to support and further develop Balancer Protocol.

## Balancer Pools

### What is a Balancer Pool?

The fundamental building blocks of Balancer Protocol are Balancer Pools. Pools are smart contracts integrated with Balancer Protocol that hold value in 2 or more ERC20 tokens. They can be seen as a market-making portfolio, since each token comprising the pool has a different value weight and can be traded against any other of its tokens. An example of pool would be one with 50% WETH, 25% MKR and 25% DAI.

Two main properties make Balancer Pools special:

1. They ensure that, even as the relative unit prices of its tokens vary, they are continuously rebalanced to maintain the desired value distribution across tokens constant. 
2. Each trade that takes place in a Balancer Pool generates a fee to the pool owner. The fee is a percentage of the amount traded, and is customizable by the pool owner. 

### Are there constraints for setting up a Balancer Pool?

Only a few. Balancer Protocol limits pools in the following ways:

* Maximum number of tokens: there can only be at most 8 different tokens in the pool 
* Token weights: the maximum weight ratio between any two tokens is 100 
* Pool fee: the fee can be chosen to be any number between 0.01% and 10% 
* ERC20 compliance: pool tokens have to be ERC20 compliant 

### How are Balancer Pools continuously rebalanced?

This is probably the main innovation brought by Balancer Protocol. Pools are efficiently rebalanced thanks to a multi-dimensional invariant function used to continuously define swap prices between any two tokens in any pool.

Imagine a portfolio that contains a proportion of token A and its price increases. A conventional rebalancing process would sell this token and consequently pay a fee for that trade. Balancer Protocol, instead, lets rational market actors actively buy token A from the pool. So instead of selling token A and paying a fee for that, the Balancer Pool let other actors buy token A and charge a fee for that.

In summary, instead of paying fees to rebalance, Balancer Pools earn fees to let others rebalance them. For further technical details, please refer to our [white paper](http://a).

### How do Balancer Pools charge fees and how much are they?

Balancer Protocol charges a percentage of the input amount traded for each trade against a Balancer Pool. The fee goes entirely to the Balancer Pool owner. See “Are there constraints for setting up a Balancer Pool?” for details about fee constraints.

### What types of Balancer pools are there?

Balancer Pools can be joinable or private. xxx

## Balancer Token Wrappers

### What is a Balancer Token Wrapper?

All ERC20 tokens used in Balancer Protocol are wrapped for simplifying usability and optimizing gas costs within the protocol. Tokens are wrapped into a bERC20 wrapper inside Balancer Protocol. All bTokens are fully collateralized, so for every bERC20 minted there is one ERC20 token in Balancer Vaults.

### Are Balancer Tokens ERC20 themselves?

Yes, all bTokens are ERC20 tokens themselves.

### When trading, can I choose to receive the actual ERC20 as opposed to a bERC20?

Whenever a trader swaps two tokens using Balancer protocol, they have the choice to receive the token they are buying in the form of a Balancer Token \(bToken\) or as the actual ERC20. The advantages of receiving and keeping bTokens are mainly the following:

* Further interactions with Balancer Protocol - like creating pools or trading - are much more gas efficient since no external ERC20 calls need to be done 
* When redeeming bTokens for the actual ERC20 users pay a small fee of 0.1% 

### When withdrawing tokens from my pool, can I choose to receive the actual ERC20 as opposed to a bERC20?

Similarly to the answer above, tokens can be withdrawn from Balancer Pools as the actual ERC20 tokens or as wrapped Balancer tokens \(bTokens\). The same advantages mentioned above apply if you withdraw pool tokens in the form of bTokens.

## Using Balancer Protocol

### How can I use Balancer as a trader?

There are three main ways a trader can interact with Balancer Protocol:

* Through our [trader front-end](http://a) 
* Calling our [API](http://a) 
* Directly through our [Smart Contracts on Ethereum mainnet](http://a) 

### How can I use Balancer as a liquidity provider or portfolio manager?

There are three main ways a liquidity provider or portfolio manager can interact with Balancer Protocol:

* Through our [pool manager front-end](http://a) 
* Directly through our [Smart Contracts on Ethereum mainnet](http://a) 

Please [contact us](http://a) if you have a portfolio worth more than 100,000 USD and need special assistance.

### Can Balancer be used as an Index Fund?

Balancer Protocol is ideal for Index Funds as it takes away all the hassle that managers have regarding rebalancing and on top of that generates fees from trades it enables.

## Miscellaneous

### Does using Balancer Protocol generate taxable events?

We cannot provide tax or accounting advice. Tax regulations are specific to jurisdiction where you or your company reside. For any legal or tax matters we recommend consulting your own attorney.

### Are there risks in using Balancer Protocol?

Balancer Protocol smart contracts have been designed having security as its top priority. We had our code be audited by best in class professionals. You can see the results of our audits [here](http://a). However, we cannot guarantee that bugs won’t be found in the future. If you use Balancer Protocol you are agreeing to our [terms & conditions](http://a).

