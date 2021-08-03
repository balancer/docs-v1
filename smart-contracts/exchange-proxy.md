# Exchange Proxy

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/exchange-proxy)

## Exchange Proxy

### Summary

This contract includes swap forwarding proxy logic and on-chain smart order routing functionality.

`batchSwap` functions allows users to batch execute swaps recommended by [off-chain SOR](sor/development.md).

`viewSplit` functions query the [On Chain Registry](on-chain-registry.md) to provide best swap information using on-chain data.

`smartSwap` functions combine view and batch functionality to provide complete optimised on-chain swaps.

### API

#### **Batch Swap Functions**

**multihopBatchSwapExactIn**

`multihopBatchSwapExactIn(Swap[][] memory swapSequences, TokenInterface tokenIn, TokenInterface tokenOut, uint totalAmountIn, uint minTotalAmountOut) public payable`

Execute multi-hop swaps returned from off-chain SOR for swapExactIn trade type.

**multihopBatchSwapExactOut**

`multihopBatchSwapExactOut(Swap[][] memory swapSequences, TokenInterface tokenIn, TokenInterface tokenOut, uint maxTotalAmountIn) public payable`

Execute multi-hop swaps returned from off-chain SOR for swapExactOut trade type.

**batchSwapExactIn**

`batchSwapExactIn(Swap[] memory swaps, TokenInterface tokenIn, TokenInterface tokenOut, uint totalAmountIn, uint minTotalAmountOut) public payable`

Execute single-hop swaps for swapExactIn trade type. Used for swaps returned from viewSplit function and legacy off-chain SOR.

**batchSwapExactOut**

`batchSwapExactOut(Swap[] memory swaps, TokenInterface tokenIn, TokenInterface tokenOut, uint maxTotalAmountIn) public payable`

Execute single-hop swaps for swapExactOut trade type. Used for swaps returned from viewSplit function and legacy off-chain SOR.

#### **View Split Functions**

**viewSplitExactIn**

`viewSplitExactIn(address tokenIn, address tokenOut, uint swapAmount, uint nPools)`

View function that calculates most optimal swaps \(exactIn swap type\) across a max of nPools. Returns an array of Swaps and the total amount out for swap.

**viewSplitExactOut**

`viewSplitExactOut(address tokenIn, address tokenOut, uint swapAmount, uint nPools)`

View function that calculates most optimal swaps \(exactOut swap type\) across a max of nPools. Returns an array of Swaps and the total amount in for swap.  
\(! Please be aware the return parameter "totalOutput" in the contract is a misnomer and actually represents totalInput !\)

#### **Smart Swap Functions**

**smartSwapExactIn**

`smartSwapExactIn(TokenInterface tokenIn, TokenInterface tokenOut, uint totalAmountIn, uint minTotalAmountOut, uint nPools) public payable`

Calculates and executes most optimal swaps across a max of nPools for tokenIn &gt; tokenOut swap with an input token amount = totalAmountIn.

**smartSwapExactOut**

`smartSwapExactOut(TokenInterface tokenIn, TokenInterface tokenOut, uint totalAmountOut, uint maxTotalAmountIn, uint nPools) public payable`

Calculates and executes most optimal swaps across a max of nPools for tokenIn &gt; tokenOut swap with a desired output token amount = totalAmountOut.

