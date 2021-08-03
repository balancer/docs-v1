# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/sor)

# SOR

## Overview

Smart Order Router, or SOR, is an off-chain linear optimization of routing orders across pools for best price execution. It takes as input a desired amount of any token to be traded for another token, and returns a list of pools/amounts that should be traded such that the amount of returned tokens is maximized. The sum of the amounts to be traded with each pool equals the desired amount given as input.

SOR exists in the Bronze release as a way to aggregate liquidity across all Balancer pools. Future releases of Balancer will accomplish this on-chain, and allow aggregate contract fillable liquidity.

Liquidity aggregators are free to use the SOR npm package or create their own order routing across pools.

## Motivation

Since there might be many Balancer pools that contain a given pair, trading only with the most liquid pool is not ideal. The fact that after a single-pool trade arbitrageurs can profit by leveling all pool prices means that the single-pool trader left value on the table.

The ideal solution - in a world where there were no gas costs or gas limits - would be to trade with all pools available in the desired pair. The amount traded with each pool should move them all to the same new spot price. This would mean all pools would be in an arbitrage-free state, and no further value could be extracted by arbitrageurs \(which always comes at the expense of the trader\).

As there are incremental gas costs associated with interacting with additional Balancer pools, the SOR ensures that any additional interactions are profitable; i.e., the profit obtained through trading with each new pool exceeds the associated gas cost. This is an optimization problem that depends on the gas fees paid to the Ethereum network: higher gas prices may cause the SOR to select fewer trading pools.

## How it works

The fundamental optimization problem is to find the path through a set of Balancer Pools with the highest net yield after gas costs. The algorithm continues to add pools to the trading set, until there are no pools left where the net gain from trading with the pool would exceed the gas cost.

### Linearization

Even though the SOR runs totally off-chain, it has been designed to be EVM-tractable, since we intend to release an on-chain version in the future.

Since we know the price will increase after trading, due to slippage, and in order to make the SOR EVM-tractable, the function that calculates the spot price of a Balancer pool after a trade has been linearized. We refer to EP – the estimated price – as the spot price a pool will have after a trade, according to that linearized approximation. The figure below shows the real \(non-linear\) spot price after a given amount is traded versus the linearized approximation we use.

![](../.gitbook/assets/picture1.png)

### Prices of interest

Since we linearize the spot price functions of all Balancer pools, we can interpolate prices and amounts to make our optimization solution simpler. 

To help visualize what this means, imagine three Balancer pools containing the trade tokens. We define EPs of interest as sets of prices where either of the following conditions apply:

1. There is a pool with exactly that initial spot price; or
2. The spot prices of two pools "cross" at that price.

The figure below shows the six different EPs of interest for that case.

![](../.gitbook/assets/picture2.png)

### Solution example

To reduce clutter and simplify the visualization, we consider only pools 1 and 2 for the following walk through.

![](../.gitbook/assets/picture3.png)

Let _A_ be the amount of _token in_  \(Ai\) to be traded on pool 1 such that its price increases from EP1 \(initial spot price of pool 1\) to EP2 \(initial spot price of pool 2\). The SOR's solution for any amount of tokens lower than _A_ is simply: "trade the entire amount with pool 1."

When the amount traded is greater than _A,_ SOR will start including pool 2 in the solution, as not doing so would mean the trader is trading some amount \(Ai - _A_\) for a higher price than they could with pool 2.

The solution for trading an amount B + C can be found by interpolating the trades that result in EP2 and EP3. By trading C with pool 1 and B with pool 2, both pools end at the same price \(Final Price\) which means that the best solution was found. 

## Development

Balancer labs have developed the [SOR npm package](https://www.npmjs.com/package/@balancer-labs/sor), an easy to use implementation of the above concepts. For documentation for working with the package please see this [page](https://docs.balancer.finance/sor/development).



