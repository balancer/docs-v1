# Perpetual Synthetic Pool

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/smart-pool-templates-gui/perpetual-synthetic-pool)

## Perpetual Synthetic Pool

Synthetic tokens, through the efforts of [UMA](https://umaproject.org/) and others, are coming into their own, grabbing the spotlight and bringing a type of derivative familiar from the traditional financial markets to crypto.

There are already balancer pools featuring synthetics \(I would link to one, but it would be out-of-date in a month.\) One [UMA synthetic token](https://medium.com/uma-project/priceless-synthetic-tokens-f28e6452c18b) current as of this writing is yUSD-OCT20.

Note that when UMA supports perpetual synthetics natively - tokens with no expiration date - this gets easier. At that point, you would not need to add or remove tokens monthly, and could simply hold them in a pool, perhaps rebalancing periodically.

Current priceless synthetics have an expiration date, and the most common implementations expire monthly. It is now common practice to create a standard BPool with a synthetic expiring a month or two in the future, and a collateral token like USDC. When the synthetic expires, people generally withdraw liquidity from the current month's pool, and add it to the next month. One of the issues with this is the gas costs incurred by all these rollovers. Your investment would need to be big enough to still profit, after fees.

One solution would be a "rolling synthetic" Smart Pool. There are many possible implementations. One is a 2-token pool with the current synthetic and a collateral coin. Near the end of the month, the pool operator would add the next month's token, and then, some time after expiration, remove the previous token.

Note that the pool creator would need to have enough pool tokens to be able to call `removeToken` \(which burns pool tokens\). The bigger the total supply, they more they need, so if a lot of other LPs joined, the creator might need to deposit more liquidity in order to swap out synthetics.

Alternatively, a pool operator could create a pool with two synthetics - the current and next month's tokens. Initially the weights would favor the current month, but over the course of the month \(tracking "implied volatility"\), the weights would "flip," so that the situation was reversed near the end of the month, with next month's token weighted highly, and the current month's heavily discounted.

These could even be combined, adding the future month's synthetic a bit in advance, and removing the expired month's synthetic after a redemption period.

Legend**: required;** ~~not required;~~ _optional_

**Rights configuration:**

* **canPauseSwapping**
* ~~canChangeSwapFee~~
* **canChangeWeights**
* _canAddRemoveTokens_
* ~~canWhitelistLPs~~
* ~~canChangeCap~~

You need to change weights, and probably also add/remove tokens, unless you make a new pool each month. If you're adding and removing tokens, you would probably want to be able to pause swapping during these operations, or during redemption periods.

