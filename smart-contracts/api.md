# API

### API Index

| Function |
| :--- |
| [`swap_ExactAmountIn(TI,AI,TO,MO,MP)->(AO,NP)`](api.md#swap_exactamountin) |
| [`swap_ExactAmountOut(TI,MI,TO,AO,MP)->(AI,NP)`](api.md#swap_exactamountout) |
| [`swap_ExactMarginalPrice(TI,MI,TO,MO,MP)->(AI,AO)`](api.md#swap_exactmarginalprice) |
| [`joinPool(PO)`](api.md#joinpool) |
| [`exitPool(PO)`](api.md#exitpool) |
| [`joinswap_ExternAmountIn(TI, AI)->(PO)`](api.md#joinswap_externamountin) |
| [`exitswap_ExternAmountOut(TO, AO)->(PI)`](api.md#joinswap_externamountout) |
| [`joinswap_PoolAmountOut(PO,TI)->(AI)`](api.md#joinswap_poolamountout) |
| [`exitswap_PoolAmountIn(PI,TO)->(AO)`](api.md#joinswap_poolamountin) |
| - |
| [`getSpotPrice(TI,TO)->(P)`](api.md#getspotprice) |
| [`getSpotRate(TI,TO)->(R)`](api.md#getspotprice) |
| [`getSpotPriceSansFee(TI,TO)->(P)`](api.md#getspotprice) |
| [`getSpotRateSansFee(TI,TO)->(R)`](api.md#getspotprice) |
| [`getFees()->(swapFee,exitFee)`](api.md#getfees) |
| - |
| [`bind(T, balance, denorm)`](api.md#bind) |
| [`rebind(T, balance, denorm)`](api.md#rebind) |
| [`unbind(T)`](api.md#unbind) |
| [`setPublicSwap(bool)`](api.md#setPublicSwap) |
| [`setPublicJoin(bool)`](api.md#setPublicJoin) |
| [`setPublicExit(bool)`](api.md#setPublicExit) |
| [`setFees(swapFee, exitFee)`](api.md#setfees) |
| [`collect()`](api.md#collect) |
| [`finalize()`](api.md#finalize) |
| [`gulp(T)`](api.md#gulp) |
| - |
| [`isFinalized()->bool`](api.md#isfinalized) |
| [`isBound()->bool`](api.md#isbound) |
| [`getBalance()->bool`](api.md#getbalance) |
| [`getNormalizedWeight(T)->(NW)`](api.md#getnormalizedweight) |
| [`getDenormalizedWeight(T)->(DW)`](api.md#getdenormalizedweight) |
| [`getTotalDenormalizedWeight()->(TDW)`](api.md#gettotaldenormalizedweight) |
| [`getNumTokens()->(N)`](api.md#getnumtokens) |
| [`getCurrentTokens()->[Ts]`](api.md#getnumtokens) |
| [`getFinalTokens->[Ts]`](api.md#getnumtokens) |
| [`getController()->address`](api.md#getcontroller) |
| - |

## Functions

## `swap_ExactAmountIn`

```text
swap_ExactAmountIn( address tokenIn, uint amountIn
                  , address tokenOut, uint minOut
                  , uint maxPrice)
    returns (uint amountOut, uint newPrice);
```

Trades an exact `amountIn` of `tokenIn` taken from the caller by the pool, in exchange for at least `minOut` of `tokenOut` given to the caller from the pool, with a maximum marginal price of `maxPrice`.

Returns `(amountOut, newPrice)`, where `amountOut` is the amount of token that came out of the pool, and MP is the new marginal spot price, ie, the result of `getSpotPrice` after the call. \(These values are what are limited by the arguments; you are guaranteed `amountOut >= minOut` and `newPrice <= maxPrice`\).

## `swap_ExactAmountOut`

```text
swap_ExactAmountOut( address tokenIn, uint maxIn
                   , address tokenOut, uint amountOut
                   , uint maxPrice)
    returns (uint amountIn, uint newPrice)
```

## `swap_ExactMarginalPrice`

```text
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

Specify `tAo` of token `T` that you want to get out of the pool. This costs `pAi` pool shares \(these went into the pool\).

## `joinswap_PoolAmountOut`

`joinswap_PoolAmountOut(uint pAo, address T) -> (uint tAi)`

Specify `pAo` pool shares that you want to get, and a token `T` to pay with. This costs `tAi` tokens \(these went into the pool\).

## `exitswap_PoolAmountIn`

`exitswap_PoolAmountIn(uint pAi, address T) -> (uint tAo)`

Pay `pAi` pool shares into the pool, getting `tAo` of the given token `T` out of the pool.

## `isFinalized`

`isFinalized() -> (bool)`

The `finalized` state lets users know that the weights, balances, and fees of this pool are immutable. In the `finalized` state, `SWAP`, `JOIN`, and `EXIT` are public. [`CONTROL` capabilities](api.md#access-control) are disabled.

## `isBound`

`isBound(address T) -> (bool)`

A bound token has a valid balance and weight. A token cannot be bound without valid parameters which will enable e.g. `getSpotPrice` in terms of other tokens. However, disabling `isSwapPublic` and `isJoinPublic` will disable any interaction with this token in practice \(assuming there are no existing tokens in the pool, which can always `exitPool`\). This is one of many factors that could lead to `getSpotPrice` being an unreliable price sensor.

## `getNumTokens`

`getNumTokens() -> (uint)`

How many tokens are bound to this pool.

## `getController`

`getController() -> (address)`

Get the "controller" address, which can call `CONTROL` functions like `rebind`, `setFees`, or `finalize`.

## `bind`

`bind(address T, uint B, uint W)`

Binds the token with address `T`. Tokens will be pushed/pulled from caller to adjust match new balance. Token must not already be bound. `B` must be a valid balance and `W` must be a valid _denormalized_ weight.

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

Makes `isPublicSwap` return `isPublic` Requires caller to be controller and pool not to be finalized. Finalized pools always have public swap.

### `setPublicJoin`

`setPublicJoin(bool p)`

Makes `isPublicJoin` return `isPublic`. Requires caller to be controller and pool not to be finalized. Finalized pools always have public join.

### `setPublicExit`

Throws `ERR_EXIT_ALWAYS_PUBLIC` if called with `p == false`. `EXIT` functions are always public on Bronze pools.

### `finalize`

`finalize()`

This makes the pool **finalized**. This is a one-way transition. `bind`, `rebind`, `unbind`, `setFees` and `setPublic<Swap/Join/Exit>` will all throw `ERR_IS_FINALIZED` after pool is finalized. This also switches `isSwapPublic` and `isJoinPublic` to true. `isExitPublic` is always true.

### `setFees`

`setFees(uint swapFee, uint exitFee)`

Caller must be controller. Pool must NOT be finalized.

