# Core Concepts

## Terminology

* Pool: A `BPool` contract object
* Balance: The total token balance of a pool. Does not refer to any user balance.
* Denorm: Denormalized weight. Weights on a BPool are configured and stored in their **denormalized** form.
* Controller: The pool's "owner"; an address that can call `CONTROL` capabilities.
* Factory: The official BPool factory. Pools deployed from this factory appear in tools and explorers for Balancer.

## Pool Lifecycle

Any user can create a new pool by calling `newBPool()` on the `BFactory` contract. The caller is set as the `controller` or pool owner.

Pools can exist in one of two states: `controlled` or `finalized`. Pools start in a controlled state and the controller may choose to make the pool finalized by calling `finalize()`. Finalize is a one-way transition. While in a controlled state, outside actors cannot add liquidity. A controlled state allows the controller to set the pool's tokens and weights.

### Smart Contract Owned Controlled Pools

One very powerful feature of Balancer is the concept of smart-contract owned controlled pools. A smart-contract controlled pool can fully emulate a finalized pool, while also allowing complex logic to readjust balances, weights, and fees. Some examples include:

* [Interest bearing stablecoin pool without impermanent loss](https://medium.com/balancer-protocol/zero-impermanent-loss-stablecoin-pool-with-lending-interests-a3da6d8bb782)
* Pool that adjusts swap fees as a function of the volatility of the pool's assets
* Updating weights to deploy a certain market-opinionated strategy
* More complex [dynamic strategies](https://caia.org/sites/default/files/dynamic_strategies_for_asset_allocation.pdf) for asset allocation

#### Conceptual Capabilities:

* `SWAP`  \(`swap_*`, `joinswap_*`, `exitswap_*`\)
* `JOIN` \(`joinPool`, `joinswap_*`\)
* `EXIT` \(`exitPool`, `exitswap_*`\)
* `CONTROL` \(`bind`, `unbind`, `rebind`, `setSwapFee`, `finalize`\)

Notice that e.g. `joinswap` requires both `JOIN` and `SWAP`.

#### Token-specific state:

* A token can be "bound" or not -- if it's bound, it has a valid balance, denorm, and can be interacted with, assuming your address has a role with the right capabilities

