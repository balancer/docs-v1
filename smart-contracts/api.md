# API

## API Index

| Function |
| :--- |
| [`swap_ExactAmountIn(tokenIn,tokenAmountIn,tokenOut,minAmountOut,maxPrice)->(tokenAmountOut,spotPriceTarget)`](api.md#swap_exactamountin) |
| [`swap_ExactAmountOut(tokenIn,maxAmountIn,tokenOut,tokenAmountOut,maxPrice)->(tokenAmountIn,spotPriceTarget)`](api.md#swap_exactamountout) |
| [`swap_ExactMarginalPrice(tokenIn,minAmountIn,tokenOut,maxAmountOut,marginalPrice)->(tokenAmountIn,tokenAmountOut)`](api.md#swap_exactmarginalprice) |
| [`joinPool(poolAmountOut)`](api.md#joinpool) |
| [`exitPool(poolAmountIn)`](api.md#exitpool) |
| [`joinswap_ExternAmountIn(tokenIn, tokenAmountIn)->(poolAmountOut)`](api.md#joinswap_externamountin) |
| [`exitswap_ExternAmountOut(tokenOut, tokenAmountOut)->(poolAmountIn)`](api.md#joinswap_externamountout) |
| [`joinswap_PoolAmountOut(poolAmountOut,tokenIn)->(tokenAmountIn)`](api.md#joinswap_poolamountout) |
| [`exitswap_PoolAmountIn(poolAmountIn,tokenOut)->(tokenAmountOut)`](api.md#joinswap_poolamountin) |
| - |
| [`getSpotPrice(tokenIn,tokenOut)->(spotPrice)`](api.md#getspotprice) |
| [`getSpotPriceSansFee(tokenIn,tokenOut)->(spotPrice)`](api.md#getspotprice) |
| [`getSwapFee()->(swapFee)`](api.md#getswapfee) |
| - |
| [`bind(token, balance, denorm)`](api.md#bind) |
| [`rebind(token, balance, denorm)`](api.md#rebind) |
| [`unbind(token)`](api.md#unbind) |
| [`setPublicSwap(bool)`](api.md#setPublicSwap) |
| [`setSwapFee(swapFee)`](api.md#setswapfee) |
| [`collect()`](api.md#collect) |
| [`finalize()`](api.md#finalize) |
| [`gulp(token)`](api.md#gulp) |
| - |
| [`isFinalized()->bool`](api.md#isfinalized) |
| [`isBound()->bool`](api.md#isbound) |
| [`getBalance()->bool`](api.md#getbalance) |
| [`getNormalizedWeight(token)->(normalizedWeight)`](api.md#getnormalizedweight) |
| [`getDenormalizedWeight(token)->(denormWeight)`](api.md#getdenormalizedweight) |
| [`getTotalDenormalizedWeight()->(totalDenormWeight)`](api.md#gettotaldenormalizedweight) |
| [`getNumTokens()->(N)`](api.md#getnumtokens) |
| [`getCurrentTokens()->[Ts]`](api.md#getnumtokens) |
| [`getFinalTokens->[Ts]`](api.md#getnumtokens) |
| [`getController()->address`](api.md#getcontroller) |
| - |

## Functions

### Controller and View Functions

#### `isBound`

`isBound(address T) -> (bool)`

A bound token has a valid balance and weight. A token cannot be bound without valid parameters which will enable e.g. `getSpotPrice` in terms of other tokens. However, disabling `isSwapPublic` will disable any interaction with this token in practice \(assuming there are no existing tokens in the pool, which can always `exitPool`\). This is one of many factors that could lead to `getSpotPrice` being an unreliable price sensor.

#### `getNumTokens`

`getNumTokens() -> (uint)`

How many tokens are bound to this pool.

#### `getController`

`getController() -> (address)`

Get the "controller" address, which can call `CONTROL` functions like `rebind`, `setSwapFee`, or `finalize`.

#### `bind`

`bind(address T, uint B, uint W)`

Binds the token with address `T`. Tokens will be pushed/pulled from caller to adjust match new balance. Token must not already be bound. `B` must be a valid balance and `W` must be a valid _denormalized_ weight.

Possible errors:

* `ERR_NOT_CONTROLLER` -- caller is not the controller
* `ERR_IS_BOUND` -- `T` is already bound
* `ERR_IS_FINALIZED` -- `isFinalized()` is true
* `ERR_ERC20_FALSE` -- `ERC20` token returned false
* unspecified error thrown by token

#### `rebind`

`rebind(address T, uint B, uint W)`

Changes the parameters of an already-bound token. Performs the same validation on the parameters.

#### `unbind`

`rebind(address T)`

Unbinds a token, clearing all of its parameters and pushing the entire balance to caller.

#### `setPublicSwap`

`setPublicSwap(bool isPublic)`

Makes `isPublicSwap` return `isPublic` Requires caller to be controller and pool not to be finalized. Finalized pools always have public swap.


#### `finalize`

`finalize()`

This makes the pool **finalized**. This is a one-way transition. `bind`, `rebind`, `unbind`, `setSwapFee` and `setPublicSwap` will all throw `ERR_IS_FINALIZED` after pool is finalized. This also switches `isSwapPublic` to true.

#### `setSwapFee`

`setSwapFee(uint swapFee)`

Caller must be controller. Pool must NOT be finalized.

#### `isFinalized`

`isFinalized() -> (bool)`

The `finalized` state lets users know that the weights, balances, and fees of this pool are immutable. In the `finalized` state, `SWAP`, `JOIN`, and `EXIT` are public. [`CONTROL` capabilities](api.md#access-control) are disabled.

### Trading and Liquidity Functions

#### `swap_ExactAmountIn`

```text
swap_ExactAmountIn(
    address tokenIn,
    uint tokenAmountIn,
    address tokenOut,
    uint minAmountOut,
    uint maxPrice
)
    returns (uint tokenAmountOut, uint spotPriceTarget)
```

Trades an exact `tokenAmountIn` of `tokenIn` taken from the caller by the pool, in exchange for at least `minAmountOut` of `tokenOut` given to the caller from the pool, with a maximum marginal price of `maxPrice`.

Returns `(tokenAmountOut, spotPriceTarget)`, where `tokenAmountOut` is the amount of token that came out of the pool, and MP is the new marginal spot price, ie, the result of `getSpotPrice` after the call. \(These values are what are limited by the arguments; you are guaranteed `tokenAmountOut >= minAmountOut` and `spotPriceTarget <= maxPrice`\).

#### `swap_ExactAmountOut`

```text
swap_ExactAmountOut(
    address tokenIn,
    uint maxAmountIn,
    address tokenOut,
    uint minAmountOut,
    uint maxPrice
)
    returns (uint tokenAmountIn, uint spotPriceTarget)
```

#### `swap_ExactMarginalPrice`

```text
swap_ExactMarginalPrice(
    address tokenIn,
    uint maxAmountIn,
    address tokenOut,
    uint minAmountOut,
    uint marginalPrice
)
    return (uint tokenAmountIn, uint tokenAmountOut)`
```

#### `joinPool`

`joinPool(uint poolAmountOut)`

Join the pool, getting `poolAmountOut` pool tokens. This will pull some of each of the currently trading tokens in the pool, meaning you must have called `approve` for each token for this pool.

#### `exitPool`

`exitPool(uint poolAmountIn)`

Exit the pool, paying `poolAmountIn` pool tokens and getting some of each of the currently trading tokens in return.

#### `joinswap_ExternAmountIn`

`joinswap_ExternAmountIn(address tokenIn, uint tokenAmountIn) -> (uint poolAmountOut)`

Pay `tokenAmountIn` of token `tokenIn` to join the pool, getting `poolAmountOut` of the pool shares.

#### `exitswap_ExternAmountOut`

`exitswap_ExternAmountOut(address tokenOut, uint tokenAmountOut) -> (uint poolAmountIn)`

Specify `tokenAmountOut` of token `tokenOut` that you want to get out of the pool. This costs `poolAmountIn` pool shares \(these went into the pool\).

#### `joinswap_PoolAmountOut`

`joinswap_PoolAmountOut(uint poolAmountOut, address tokenIn) -> (uint tokenAmountIn)`

Specify `poolAmountOut` pool shares that you want to get, and a token `tokenIn` to pay with. This costs `tokenAmountIn` tokens \(these went into the pool\).

#### `exitswap_PoolAmountIn`

`exitswap_PoolAmountIn(uint poolAmountIn, address tokenOut) -> (uint tokenAmountOut)`

Pay `poolAmountIn` pool shares into the pool, getting `tokenAmountOut` of the given token `tokenOut` out of the pool.





