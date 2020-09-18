# Development

Documentation for working with the `@balancer-labs/sor` package. For a description of the SOR and math, see this [page](../protocol/sor.md#overview).

{% hint style="danger" %}
Please take caution as the SOR is under heavy development and may have breaking changes.
{% endhint %}

The SOR package includes a primary `smartOrderRouter` function, along with several helper functions for working with the subgraph and formatting data.

## Subgraph

All numerical data such as balances, weights, fees, etc. stored in subgraph are stored in normalized decimal point form. This is to allow for easy integrations with partners and teams building on top of Balancer.

The network subgraph can be chosen using the environment variable: `REACT_APP_SUBGRAPH_URL`. It currently defaults to Mainnet, which is: [https://api.thegraph.com/subgraphs/name/balancer-labs/balancer](https://api.thegraph.com/subgraphs/name/balancer-labs/balancer)

Several helper functions exist for common queries against the subgraph:

#### **`getPoolsWithTokens(tokenIn, tokenOut)`**

This returns a list of all pools in the balancer ecosystem that include the pair: `tokenIn` & `tokenOut` and also have `publicSwap` enabled

#### `parsePoolData(pools, tokenIn, tokenOut): Pool[]`

This function formats all the returned pool data from `getPoolsWithTokens` into a format compatible with the SOR. It includes scaling balances to their wei representations based on token decimals, and converting denormalized weights into normalized weights. Returns an array of type Pool:

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

#### `parsePoolDataOnChain(pools, tokenIn, tokenOut, multiAddress, provider): Pool[]`

This function is the same as `parsePoolData`, but pulls all balances, weights, and fees on-chain using [multicall](https://github.com/makerdao/multicall). This requires a web3 provider \(ex: local node or Infura\) that can be passed using `provider`. `multiAddress` is the address for the deployed multicall contract - on Mainnet this is `0xeefba1e63905ef1d7acba5a8513c70307c1ce441`

## smartOrderRouter

The `smartOrderRouter` function takes in pool data and trade parameters and performs an optimization for the best price execution.

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

`swapType` - string: either `swapExactIn` or `swapExactOut`

`targetInputAmount` -  amount to be traded, in Wei

`maxBalancers` - max number of pools to split the trade across

`costOutputToken` - the cost of the output token in ETH multiplied by the gas cost to perform the swap. This is used to determine whether the lower price obtained through including an additional pool in the transaction outweigh the gas costs. We are working to have helper data for this parameter. In the meantime, we recommend setting this to `0`, and limiting the number of `maxBalancers` to a reasonable number given gas costs.

## Example

Below is an example snippet that uses most helper functions, along with order routing, to return a final list of swaps and the expected output. The `swaps` returned can then be passed on to the exchange proxy or otherwise used to atomically execute the trades.

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

## Example using onchain balances

Below is the same example as above, but using `parsePoolDataOnChain`

```javascript
const sor = require('@balancer-labs/sor');
const BigNumber = require('bignumber.js');
const ethers = require('ethers');


const MAX_UINT = ethers.constants.MaxUint256;

const providerUrl = 'http://localhost:8545' // can use infura, etc.
const provider = new ethers.providers.JsonRpcProvider(providerUrl);

// MAINNET
const multi = '0xeefba1e63905ef1d7acba5a8513c70307c1ce441'
const tokenIn = '0x6B175474E89094C44Da98b954EedeAC495271d0F' // DAI
const tokenOut = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2' // WETH


(async function() {
    const data = await sor.getPoolsWithTokens(tokenIn, tokenOut);

    const poolData = await sor.parsePoolDataOnChain(data.pools, tokenIn, tokenOut, multi, provider));

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

