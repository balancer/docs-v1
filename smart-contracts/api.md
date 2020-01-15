# API

## API Index

| Function |
| :--- |
| [`swapExactAmountIn(tokenIn, tokenAmountIn, tokenOut, minAmountOut, maxPrice)->(tokenAmountOut, spotPriceAfter)`](api.md#swapexactamountin) |
| [`swapExactAmountOut(tokenIn, maxAmountIn, tokenOut, tokenAmountOut, maxPrice)->(tokenAmountIn, spotPriceAfter)`](api.md#swapexactamountout) |
| [`joinPool(poolAmountOut, [maxAmountsIn])`](api.md#joinpool) |
| [`exitPool(poolAmountIn, [minAMountsOut])`](api.md#exitpool) |
| [`joinswapExternAmountIn(tokenIn, tokenAmountIn, minPoolAmountOut)->(poolAmountOut)`](api.md#joinswapexternamountin) |
| [`exitswapExternAmountOut(tokenOut, tokenAmountOut, maxPoolAmountIn)->(poolAmountIn)`](api.md#joinswapexternamountout) |
| [`joinswapPoolAmountOut(poolAmountOut, tokenIn, maxAmountIn)->(tokenAmountIn)`](api.md#joinswappoolamountout) |
| [`exitswapPoolAmountIn(poolAmountIn, tokenOut, minAmountOut)->(tokenAmountOut)`](api.md#joinswappoolamountin) |
| - |
| [`getSpotPrice(tokenIn,tokenOut)->(spotPrice)`](api.md#getspotprice) |
| [`getSpotPriceSansFee(tokenIn,tokenOut)->(spotPrice)`](api.md#getspotpricesansfee) |
| [`getSwapFee()->(swapFee)`](api.md#getswapfee) |
| - |
| [`bind(token, balance, denorm)`](api.md#bind) |
| [`rebind(token, balance, denorm)`](api.md#rebind) |
| [`unbind(token)`](api.md#unbind) |
| [`setPublicSwap(bool)`](api.md#setPublicSwap) |
| [`setSwapFee(swapFee)`](api.md#setswapfee) |
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

A bound token has a valid balance and weight. A token cannot be bound without valid parameters which will enable e.g. `getSpotPrice` in terms of other tokens. However, disabling `isSwapPublic` will disable any interaction with this token in practice \(assuming there are no existing tokens in the pool, which can always `exitPool`\).

#### `getNumTokens`

`getNumTokens() -> (uint)`

How many tokens are bound to this pool.

#### `getController`

`getController() -> (address)`

Get the "controller" address, which can call `CONTROL` functions like `rebind`, `setSwapFee`, or `finalize`.

#### `bind`

`bind(address token, uint balance, uint denorm)`

Binds the token with address `token`. Tokens will be pushed/pulled from caller to adjust match new balance. Token must not already be bound. `balance` must be a valid balance and `denorm` must be a valid _denormalized_ weight. `bind` creates the token record and then calls `rebind` for updating pool weights and token transfers.

Possible errors:

* `ERR_NOT_CONTROLLER` -- caller is not the controller
* `ERR_IS_BOUND` -- `T` is already bound
* `ERR_IS_FINALIZED` -- `isFinalized()` is true
* `ERR_ERC20_FALSE` -- `ERC20` token returned false
* `ERR_MAX_TOKENS` -- Only 8 tokens are allowed per pool
* unspecified error thrown by token

#### `rebind`

`rebind(address token, uint balance, uint denorm)`

Changes the parameters of an already-bound token. Performs the same validation on the parameters.

#### `unbind`

`unbind(address token)`

Unbinds a token, clearing all of its parameters. Exit fee is charged and the remaining balance is sent to caller.

#### `setPublicSwap`

`setPublicSwap(bool isPublic)`

Makes `isPublicSwap` return `_publicSwap` Requires caller to be controller and pool not to be finalized. Finalized pools always have public swap.

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

#### `swapExactAmountIn`

```text
swapExactAmountIn(
    address tokenIn,
    uint tokenAmountIn,
    address tokenOut,
    uint minAmountOut,
    uint maxPrice
)
    returns (uint tokenAmountOut, uint spotPriceAfter)
```

Trades an exact `tokenAmountIn` of `tokenIn` taken from the caller by the pool, in exchange for at least `minAmountOut` of `tokenOut` given to the caller from the pool, with a maximum marginal price of `maxPrice`.

Returns `(tokenAmountOut, spotPriceAfter)`, where `tokenAmountOut` is the amount of token that came out of the pool, and `spotPriceAfter` is the new marginal spot price, ie, the result of `getSpotPrice` after the call. \(These values are what are limited by the arguments; you are guaranteed `tokenAmountOut >= minAmountOut` and `spotPriceAfter <= maxPrice`\).

#### `swapExactAmountOut`

```text
swapExactAmountOut(
    address tokenIn,
    uint maxAmountIn,
    address tokenOut,
    uint minAmountOut,
    uint maxPrice
)
    returns (uint tokenAmountIn, uint spotPriceAfter)
```

#### `joinPool`

`joinPool(uint poolAmountOut, uint[] calldata maxAmountsIn)`

Join the pool, getting `poolAmountOut` pool tokens. This will pull some of each of the currently trading tokens in the pool, meaning you must have called `approve` for each token for this pool. These values are limited by the array of `maxAmountsIn` in the order of the pool tokens.

#### `exitPool`

`exitPool(uint poolAmountIn, uint[] calldata minAmountsOut)`

Exit the pool, paying `poolAmountIn` pool tokens and getting some of each of the currently trading tokens in return. These values are limited by the array of `minAmountsOut` in the order of the pool tokens.

#### `joinswapExternAmountIn`

`joinswapExternAmountIn(address tokenIn, uint tokenAmountIn, uint minPoolAmountOut) -> (uint poolAmountOut)`

Pay `tokenAmountIn` of token `tokenIn` to join the pool, getting `poolAmountOut` of the pool shares.

#### `exitswapExternAmountOut`

`exitswapExternAmountOut(address tokenOut, uint tokenAmountOut, uint maxPoolAmountIn) -> (uint poolAmountIn)`

Specify `tokenAmountOut` of token `tokenOut` that you want to get out of the pool. This costs `poolAmountIn` pool shares \(these went into the pool\).

#### `joinswapPoolAmountOut`

`joinswapPoolAmountOut(uint poolAmountOut, address tokenIn, uint maxAmountIn) -> (uint tokenAmountIn)`

Specify `poolAmountOut` pool shares that you want to get, and a token `tokenIn` to pay with. This costs `tokenAmountIn` tokens \(these went into the pool\).

#### `exitswapPoolAmountIn`

`exitswapPoolAmountIn(uint poolAmountIn, address tokenOut, uint minAmountOut) -> (uint tokenAmountOut)`

Pay `poolAmountIn` pool shares into the pool, getting `tokenAmountOut` of the given token `tokenOut` out of the pool.

