# Core Concepts

## Terminology

* Pool: A `BPool` contract object
* Balance: The total token balance of a pool. Does not refer to any user balance.
* Denorm: Denormalized weight. Weights on a BPool are configured and stored in their **denormalized** form.
* Controller: The pool's "owner"; an address that can call `CONTROL` capabilities.
* Factory: The official BPool factory. Pools deployed from this factory appear in tools and explorers for Balancer.

## Pool Lifecycle

Any user can create a new pool by calling `newBPool()` on the `BFactory` contract. The caller is set as the `controller` or pool owner.

Pools can exist in one of two states: `controlled` or `finalized`. Pools start in a controlled state and the controller may choose to make the pool finalized by calling `finalize()`. While in a controlled state, outside actors cannot add liqudity. A controlled state allows the controller to set the pool's tokens and weights. 

The table below illustrates pool states and the allowed calls:



### Smart Contract Owned Controlled Pools

One very powerful feature of Balancer is the concept of smart-contract owned controlled pools. 

##### Conceptual Capabilities:

* `SWAP`  (`swap_*`, `joinswap_*`, `exitswap_*`)
* `JOIN` (`joinPool`, `joinswap_*`)
* `EXIT` (`exitPool`, `exitswap_*`) <-- always public
* `CONTROL` (`bind`, `unbind`, `rebind`, `setFees`, `finalize`)
* `FACTORY` (`collect`)

Notice that e.g. `joinswap` requires both `JOIN` and `SWAP`.