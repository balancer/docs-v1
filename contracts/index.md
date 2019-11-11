# Balancer Protocol Specification

## Note about releases

The **🍂Bronze Release🍂**  is the first of 3 planned releases of the Balancer Protocol. Bronze emphasizes code clarity for audit and verification, and does not go to great lengths to optimize for gas.

The **❄️Silver Release❄️** will bring many gas optimizations and architecture changes that will reduce transaction overhead and enable more flexibile usage patterns.

The **☀️Golden Release☀️** will introduce a few final features that will tie the whole system together.

Unless noted otherwise, this document describes the contract architecture for the **🍂Bronze Release🍂**.
We have attempted to make note of the functionality that will change for future releases,
but it is possible for *any aspect* of the design to change in both very large and also very subtle ways.

Please take care to when interacting with a Balancer pool to ensure you know which release it is associated with.
Objects in the Balancer system provide `getColor() returns (bytes32)`, which returns a left-padded ASCII representation of the all-caps color word, ie, `bytes32("BRONZE")`.


##### Token-specific state:

* A token can be "bound" or not -- if it's bound, it is funded, has a valid denorm, and can be interacted with, assuming your address has a role with the right capabilities

With this mental model in mind, you can proceed to the status functions for Bronze pools.

### Pool Status

* `isPublicSwap()`
* `isPublicJoin()`
* `isPublicExit()` (always true in Bronze release)
* `isBound(address T)`
* `isFinalized()`
* `getController() returns (address C)`


| capability | when finalized is FALSE | when finalized is TRUE |
|-|-|-|
| `SWAP` | controller OR public | public |
| `JOIN` | controller OR public | public |
| `EXIT` | public | public |
| `CONTROL` | controller only | none |
| `FACTORY` | factory only | factory only |
|-|-|-|

In other words, `setPublicSwap` and `setPublicJoin` can be used to configured `SWAP` and `JOIN` functions.
`setPublicExit(false)` will throw `ERR_EXIT_ALWAYS_PUBLIC`.
`finalize` is used to make a public pool with immutable parameters.


### Events


`LOG_CALL` is an anonymous event which uses the function signature as the event signature.
It is fired by all stateful functions.

`LOG_SWAP` is fired (along with `LOG_CALL`) for all [swap variants]().

```
event LOG_SWAP( address indexed caller
              , address indexed tokenIn
              , address indexed tokenOut
              , uint256         amountIn
              , uint256         amountOut
              );

event LOG_CALL( bytes4  indexed sig
              , address indexed caller
              , bytes           data
              ) anonymous; // LOG_CALL is `anonymous` to replace default signature
```