# Development

Documentation for working with the `@balancer-labs/sor` package. For a description of the SOR and math, see this [page](../protocol/sor.md#overview).

{% hint style="danger" %}
Please take caution as the SOR is under heavy development and may have breaking changes.
{% endhint %}

The SOR package includes a primary `smartOrderRouter` function along with several helper functions for working with the subgraph and formatting data.

## Subgraph

All numerical data such as balances, weights, fees, etc. stored in subgraph are stored in a normalized decimal point form. This is to allow for easy integrations with partners and teams building on top of Balancer.

The network subgraph can be chosen using the environment variable: `REACT_APP_SUBGRAPH_URL`. It currently defaults to mainnet which is: [https://api.thegraph.com/subgraphs/name/balancer-labs/balancer](https://api.thegraph.com/subgraphs/name/balancer-labs/balancer)

Several helper functions exist for common queries against the subgraph:

#### **`getPoolsWithTokens(tokenIn, tokenOut)`**

This returns a list of all pools in the balancer ecosystem that include the pair: `tokenIn` & `tokenOut` and also have `publicSwap` enabled

#### `parsePoolData(pools, tokenIn, tokenOut): Pool[]`

This function formats all the returned pool data from `getPoolsWithTokens` into a format that the SOR needs. Includes scaling balances to their wei representation based on token decimals and converting denormalized weights into normalized weights. Returns an array of type Pool:

```javascript
export interface Pool {
    id: string;
    balanceIn: BigNumber;
    balanceOut: BigNumber;
    weightIn: BigNumber;
    weightOut: BigNumber;
    swapFee: BigNumber;
    spotPrice?: BigNumber;
    slippage?: BigNumber;
    limitAmount?: BigNumber;
}
```

## smartOrderRouter

The `smartOrderRouter` function takes in pool data and trade parameters and does an optimization for best price execution.

```javascript
export const smartOrderRouter = (
    balancers: Pool[],
    swapType: string,
    targetInputAmount: BigNumber,
    maxBalancers: number,
    costOutputToken: BigNumber
): SwapAmount[] => {
```

`balancers` - array of type `Pool` shown above

`swapType` - string of either `swapExactIn` or `swapExactOut`

`targetInputAmount` -  amount to be traded in wei form

`maxBalancers` - max number of pools to split the trade across

`costOutputToken` - the cost of the output token in ETH multiplied by the swap gas cost. This is used to determine if the gas costs of using an additional pool outweigh the better price from splitting. We are working to have helper data for this parameter. In the meantime, it is recommended to set this as `0` and limit the number of `maxBalancers` to a reasonable number given gas costs.

## Example

Below is an example snippet that uses most helper functions along with the order routing to return a final list of swaps and the expected output. The `swaps` returned can then be passed on to the exchange proxy or use another solution to atomically execute the trades.

```javascript
const sor = require('@balancer-labs/sor');
const BigNumber = require('bignumber.js');
const ethers = require('ethers');


const MAX_UINT = ethers.constants.MaxUint256;

// MAINNET
const tokenIn = '0x6B175474E89094C44Da98b954EedeAC495271d0F' // DAI
const tokenOut = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2' // WETH


(async function() {
    const data = await sor.getPoolsWithTokens(tokenIn, tokenOut);

    const poolData = sor.parsePoolData(data.pools, tokenIn, tokenOut);

    const sorSwaps = sor.smartOrderRouter(
        poolData,
        'swapExactIn',
        new BigNumber('10000000000000000000'),
        new BigNumber('10'),
        0
    );

    const swaps = sor.formatSwapsExactAmountIn(sorSwaps, MAX_UINT, 0)

    const expectedOut = sor.calcTotalOutput(swaps, poolData)
})()
```

\`\`

