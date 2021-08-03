# On Chain Registry

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/on-chain-registry)

## On Chain Registry

### Summary

Stores a registry of Balancer Pool addresses for a given token address pair. Pools can be sorted in order of liquidity and queried via view functions. Used in combination with the [Exchange Proxy](exchange-proxy.md) swaps can be sourced and exectured entirely on-chain.

### API

#### **Adding Pools To Registry**

**addPoolPair**

`addPoolPair(address pool, address token1, address token2)`

Adds a single pool address for token pair.

**addPools**

`addPools(address[] calldata pools, address token1, address token2)`

Adds an array of pool addresses for token pair.

#### **Sorting Pools**

**sortPools**

`sortPools(address[] calldata tokens, uint256 lengthLimit)`

Sorts pools in order of liquidity. lengthLimit can be used to limit the number of pools sorted.

**sortPoolsWithPurge**

`sortPoolsWithPurge(address[] calldata tokens, uint256 lengthLimit)`

Sorts pools in order of liquidity and removes any pools with &lt;10% of total liquidity.

#### **Retrieving Pools**

**getBestPools**

`getBestPools(address fromToken, address destToken)`

Retrieve array of pool addresses for token pair. Ordered by liquidity if previously sorted. Max of 32 pools returned.

**getBestPoolsWithLimit**

`getBestPoolsWithLimit(address fromToken, address destToken, uint256 limit)`

Retrieve array of pool addresses for token pair. Ordered by liquidity if previously sorted. Max of n pools returned where n=limit.

**getPoolsWithLimit**

`getPoolsWithLimit(address fromToken, address destToken, uint256 offset, uint256 limit)`

Retrieve array of pool addresses using an offset starting position.

