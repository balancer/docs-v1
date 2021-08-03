# Swing Trading Pool

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/smart-pool-templates-gui/swing-trading-pool)

## Swing Trading Pool

Recall the basic design of investment pools: traders profit through arbitrage opportunities, created when "internal" prices diverge from token market values, and pay fees for the privilege, which accrue to the liquidity providers.

There is a natural trade-off here. Low fees and high trade volume leads to impermanent loss for liquidity providers; the arbers extract more value than they're adding in fees. On the other hand, high fees discourage trading altogether, so lower rates might actually generate more total revenue.

One strategy to limit impermanent loss and maximize return is to create a pool that acts more like an index fund in traditional finance. You want something immune to day-to-day price fluctuations - so during "quiet" periods is equivalent to HODLing, since the balances don't change without trading. However, when the external market shifts significantly, the portfolio should "rebalance" through arbitrage in a manner favorable to both liquidity providers and traders. Here's a [paper](https://medium.com/balancer-protocol/high-fee-balancer-pools-for-swing-trading-8bc1c169a4c2) describing the concept.

Say you have a volatile asset \(e.g., a basket of different types of wrapped BTC\), which can swing 4-5% on a boring day at the office, but makes big 10%+ moves relatively rarely. If you created a pool of such tokens with a swap fee above the "froth" \(e.g., 7%, 8%, even the maximum 10%\), it would be static most of the time. Yet on really big moves, the pool would be guaranteed to sell high and buy low - outperforming HODLing.

Legend**: required;** ~~not required;~~ _optional_

**Rights configuration:**

* ~~canPauseSwapping~~
* _canChangeSwapFee_
* ~~canChangeWeights~~
* ~~canAddRemoveTokens~~
* ~~canWhitelistLPs~~
* ~~canChangeCap~~

You could implement this pool with no rights enabled at all \(= no trust required\). In which case you could just use a traditional BPool, and don't need a Smart Pool at all. However, you might want to change the swap fee to adjust to volatility \(e.g., if volatility increases to where daily fluctuations are triggering lots of trades\), you might want to raise the fee. Conversely, if volatility is lower than expected, you might want to lower it.

You might create an off-chain monitor that would update the fee dynamically in response to a volatility measure. Or the fee could be controlled through a DAO or less formal community voting mechanism.

