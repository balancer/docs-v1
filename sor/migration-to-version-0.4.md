# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/sor/migration-to-version-0.4)

# Migration to Version 1.0

The SOR Version 1.0.0-0 introduces breaking changes but should be fairly easy to update. Benefits of updating include - support for multihop swaps and an easier to use wrapper interface as well as more efficient/faster processing.

## Overview Of Changes

### SOR Library

The SOR is now instantiated as an object:

#### `const SOR = new sor.SOR(Provider, GasPrice, MaxPools, ChainId, PoolsUrl)`

* Find further details on parameters [here](development.md#sor-object).

SOR now has three main functions:

#### `await SOR.setCostOutputToken(tokenOut)`

* Calculates gas cost for swapping token on Balancer. 
* The result is used to make more gas efficient swap recommendations.
* Result is cached for future use but can be updated by re-calling.

#### `const isSuccess = await SOR.fetchPools()`

* This fetches all Balancer pool information and on-chain balances.
* Retrieves pool information from static IPFS file rather than Subgraph.
* Retrieves on-chain balances using custom multicall contract.
* Returns true on success or false on error.
* Information is cached and used for all future processing, this results in fast processing of swaps, i.e. useful when changing swap amounts or types in a UI.
* Accurate/valid swaps rely on upM to date balance information so it is recommended this function is re-called to refresh as needed.

#### `const [swaps, amount] = await SOR.getSwaps(tokenIn, tokenOut, swapType, swapAmount)`

* Main function to process trade. 
* Will return amount expected and a list of swaps that can be executed on-chain.

### ExchangeProxy Contract

A new ExchangeProxy contract that supports multihop swaps has been deployed to the addresses below. For contract details please see [Exchange Proxy](../smart-contracts/exchange-proxy.md).

* Mainnet: `0x3E66B66Fd1d0b02fDa6C811Da9E0547970DB2f21`
* Kovant: `0x4e67bf5bD28Dd4b570FBAFe11D0633eCbA2754Ec`

## Further Information

* For more information on SOR functions and to see code examples \(including using with ExchangeProxy\) please see [Development & Examples](development.md). 
* For a full example demonstrating use of SOR wrapper please see [here](https://github.com/balancer-labs/balancer-sor/blob/master/test/testScripts/example-simpleSwap.ts).
* There is also an [example](https://github.com/balancer-labs/balancer-sor/blob/master/test/testScripts/example-swapExactIn.ts) showing how the SOR functions can be used without the wrapper.

