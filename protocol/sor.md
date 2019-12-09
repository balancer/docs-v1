# SOR

## Overview

Smart Order Router, or SOR, is an off-chain linear optimization of routing orders across pools for best price execution.

SOR exists in the Bronze release as a way to aggregate liquidity across all Balancer pools. Future releases of Balancer will accomplish this on-chain and allow aggregate contract fillable liquidity.

Liquidity aggregators are free to use the SOR npm package or create their own order routing across pools.

## Motivation

Since there might be many Balancer pools that contain a given pair, trading only with be most liquid pool is not ideal. The fact that after a one-pool-trade arbitrageurs can profit by leveling all pool prices means the one-pool trader left value on the table. 

The ideal solution in a world where there were no gas costs or gas limits would be to trade with all pools available in the desired pair. The amount traded with each pool should move them all to the same new spot price. This would mean all pools would be in an arbitrage free state and no value could be further extracted by arbitrageurs (which always comes at the expense of the trader).

As there are incremental gas costs associated with interacting with each additional Balancer pool, SOR makes sure that the increase in the returns an incremental pool brings exceeds the incremental gas costs. This is an optimization problem that depends therefore on the gas fees paid to the ethereum network: a higher gas price may cause SOR to select less pools than otherwise.

## How it works

The fundamental optimization SOR seeks to solve is to find a set of Balancer pools to interact with such that, considering gas costs, there is no additional pool that could be added to that set such that the returns of the trade would be increased.

### Linearization
Even though SOR runs totally off-chain, it has been designed to be EVM-tractable, since our idea is to release an on-chain version in the future.

In order to make SOR EVM-tractable, the function that calculates the spot price of a Balancer pool after a given amount is traded has been linearized for our algorithm. We refer to EP -- the estimated price -- as the spot price a pool will have after a trade according to that linearized approximation. The figure below shows the real spot price after a given amount is traded (non-linear) versus its linearized approximation which we use.

xxxx

