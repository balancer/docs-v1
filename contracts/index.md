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

## Contents
* [Note about releases](#note-about-releases)
* [Contents](#contents)
* [Terminology](#terminology)
* * [Access Control](#access-control)
* * [Pool Status](#pool-status)
* [Events](#events)
* [API Index](#api-index)
* [Functions](#functions)

## Terminology

* Pool: A `BPool` contract object
* Balance: The total token balance of a pool. Does not refer to any user balance.
* Denorm: Denormalized weight. Weights on a BPool are configured and stored in their **denormalized** form.
* Controller: The pool's "owner"; an address that can call `CONTROL` capabilities.
* Factory: The official BPool factory. Pools deployed from this factory appear in tools and explorers for Balancer.

### Access Control

The access control state of a pool can be modeled with a rudimentary role-based access control system.

##### Conceptual Roles:

* factory
* controller
* public

##### Conceptual Capabilities:

* `SWAP`  (`swap_*`, `joinswap_*`, `exitswap_*`)
* `JOIN` (`joinPool`, `joinswap_*`)
* `EXIT` (`exitPool`, `exitswap_*`) <-- always public
* `CONTROL` (`bind`, `unbind`, `rebind`, `setFees`, `finalize`)
* `FACTORY` (`collect`)

Notice that e.g. `joinswap` requires both `JOIN` and `SWAP`.

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



### API Index

| Function |
|-
|[`swap_ExactAmountIn(TI,AI,TO,MO,MP)->(AO,NP)`](#swap_exactamountin)
|[`swap_ExactAmountOut(TI,MI,TO,AO,MP)->(AI,NP)`](#swap_exactamountout) |
|[`swap_ExactMarginalPrice(TI,MI,TO,MO,MP)->(AI,AO)`](#swap_exactmarginalprice) | 
|[`joinPool(PO)`](#joinpool) | 
|[`exitPool(PO)`](#exitpool) | 
|[`joinswap_ExternAmountIn(TI, AI)->(PO)`](#joinswap_externamountin) |
|[`exitswap_ExternAmountOut(TO, AO)->(PI)`](#joinswap_externamountout) |
|[`joinswap_PoolAmountOut(PO,TI)->(AI)`](#joinswap_poolamountout) |
|[`exitswap_PoolAmountIn(PI,TO)->(AO)`](#joinswap_poolamountin) |
|-
|[`getSpotPrice(TI,TO)->(P)`](#getspotprice)
|[`getSpotRate(TI,TO)->(R)`](#getspotprice)
|[`getSpotPriceSansFee(TI,TO)->(P)`](#getspotprice)
|[`getSpotRateSansFee(TI,TO)->(R)`](#getspotprice)
|[`getFees()->(swapFee,exitFee)`](#getfees) |
|-
|[`bind(T, balance, denorm)`](#bind) | 
|[`rebind(T, balance, denorm)`](#rebind) | 
|[`unbind(T)`](#unbind) | 
|[`setPublicSwap(bool)`](#setPublicSwap) |
|[`setPublicJoin(bool)`](#setPublicJoin) |
|[`setPublicExit(bool)`](#setPublicExit) |
|[`setFees(swapFee, exitFee)`](#setfees) | 
|[`collect()`](#collect) | 
|[`finalize()`](#finalize) | 
|[`gulp(T)`](#gulp) | 
|-
|[`isFinalized()->bool`](#isfinalized) | 
|[`isBound()->bool`](#isbound) | 
|[`getBalance()->bool`](#getbalance) |
|[`getNormalizedWeight(T)->(NW)`](#getnormalizedweight) | 
|[`getDenormalizedWeight(T)->(DW)`](#getdenormalizedweight) | 
|[`getTotalDenormalizedWeight()->(TDW)`](#gettotaldenormalizedweight) | 
|[`getNumTokens()->(N)`](#getnumtokens) |
|[`getCurrentTokens()->[Ts]`](#getnumtokens) |
|[`getFinalTokens->[Ts]`](#getnumtokens) |
|[`getController()->address`](#getcontroller) |
|-


## Functions

## `swap_ExactAmountIn`
```
swap_ExactAmountIn( address tokenIn, uint amountIn
                  , address tokenOut, uint minOut
                  , uint maxPrice)
    returns (uint amountOut, uint newPrice);
```

Trades an exact `amountIn` of `tokenIn` taken from the caller by the pool, in exchange for at least `minOut` of `tokenOut` given to the caller from the pool, with a maximum marginal price of `maxPrice`.

Returns `(amountOut, newPrice)`, where `amountOut` is the amount of token that came out of the pool, and MP is the new marginal spot price, ie, the result of `getSpotPrice` after the call. (These values are what are limited by the arguments; you are guaranteed `amountOut >= minOut` and `newPrice <= maxPrice`).

## `swap_ExactAmountOut`
```
swap_ExactAmountOut( address tokenIn, uint maxIn
                   , address tokenOut, uint amountOut
                   , uint maxPrice)
    returns (uint amountIn, uint newPrice)
```

## `swap_ExactMarginalPrice`
```
swap_ExactMarginalPrice( address tokenIn, uint maxIn
                       , address tokenOut, uint maxOut
                       , uint newPrice)
    return (uint amountIn, uint amountOut)`
```

## `joinPool`
`joinPool(uint shares_out)`

Join the pool, getting `shares_out` pool tokens. This will pull some of each of the currently trading tokens in the pool, meaning you must have called `approve` for each token for this pool.

## `exitPool`
`exitPool(uint shares_in)`

Exit the pool, paying `shares_in` pool tokens and getting some of each of the currently trading tokens in return.

## `joinswap_ExternAmountIn`
`joinswap_ExternAmountIn(address T, uint tAi) -> (uint pAo)`

Pay `tAi` of token `T` to join the pool, getting `pAo` of the pool shares.


## `exitswap_ExternAmountOut`
`exitswap_ExternAmountOut(address T, uint tAo) -> (uint pAi)`

Specify `tAo` of token `T` that you want to get out of the pool. This costs `pAi` pool shares (these went into the pool).

## `joinswap_PoolAmountOut`
`joinswap_PoolAmountOut(uint pAo, address T) -> (uint tAi)`

Specify `pAo` pool shares that you want to get, and a token `T` to pay with. This costs `tAi` tokens (these went into the pool).

## `exitswap_PoolAmountIn`
`exitswap_PoolAmountIn(uint pAi, address T) -> (uint tAo)`

Pay `pAi` pool shares into the pool, getting `tAo` of the given token `T` out of the pool.

## `isFinalized`
`isFinalized() -> (bool)`

The `finalized` state lets users know that the weights, balances, and fees of this pool are immutable.
In the `finalized` state, `SWAP`, `JOIN`, and `EXIT` are public. [`CONTROL` capabilities](#access-control) are disabled.

## `isBound`
`isBound(address T) -> (bool)`

A bound token has a valid balance and weight. A token cannot be bound without valid parameters which will enable e.g. `getSpotPrice` in terms of other tokens. However, disabling `isSwapPublic` and `isJoinPublic` will disable any interaction with this token in practice (assuming there are no existing tokens in the pool, which can always `exitPool`). This is one of many factors that could lead to `getSpotPrice` being an unreliable price sensor.

## `getNumTokens`
`getNumTokens() -> (uint)`

How many tokens are bound to this pool.

## `getController`
`getController() -> (address)`

Get the "controller" address, which can call `CONTROL` functions like `rebind`, `setFees`, or `finalize`.



## `bind`
`bind(address T, uint B, uint W)`

Binds the token with address `T`. Tokens will be pushed/pulled from caller to adjust match new balance.
Token must not already be bound.
`B` must be a valid balance and `W` must be a valid *denormalized* weight.

Possible errors:
* `ERR_NOT_CONTROLLER` -- caller is not the controller
* `ERR_IS_BOUND` -- `T` is already bound
* `ERR_IS_FINALIZED` -- `isFinalized()` is true
* `ERR_ERC20_FALSE` -- `ERC20` token returned false
* unspecified error thrown by token

### `rebind`
`rebind(address T, uint B, uint W)`

Changes the parameters of an already-bound token. Performs the same validation on the parameters.

### `unbind`
`rebind(address T)`

Unbinds a token, clearing all of its parameters and pushing the entire balance to caller.

### `setPublicSwap`
`setPublicSwap(bool isPublic)`

Makes `isPublicSwap` return `isPublic`
Requires caller to be controller and pool not to be finalized.
Finalized pools always have public swap.

### `setPublicJoin`
`setPublicJoin(bool p)`

Makes `isPublicJoin` return `isPublic`.
Requires caller to be controller and pool not to be finalized.
Finalized pools always have public join.

### `setPublicExit`

Throws `ERR_EXIT_ALWAYS_PUBLIC` if called with `p == false`.
`EXIT` functions are always public on Bronze pools.

### `finalize`
`finalize()`

This makes the pool **finalized**.
This is a one-way transition.
`bind`, `rebind`, `unbind`, `setFees` and `setPublic<Swap/Join/Exit>` will all throw `ERR_IS_FINALIZED` after pool is finalized.
This also switches `isSwapPublic` and `isJoinPublic` to true. `isExitPublic` is always true.

### `setFees`
`setFees(uint swapFee, uint exitFee)`

Caller must be controller.
Pool must NOT be finalized.