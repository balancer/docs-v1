# Development & Examples

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/sor/development)

## Development & Examples

Documentation for working with the `@balancer-labs/sor` package. For a description of the SOR and math, see this [page](./#overview).

{% hint style="danger" %}
Please take caution as the SOR is under heavy development and may have breaking changes.
{% endhint %}

The SOR package includes a primary `SOR` object with an `SOR.getSwaps` function and several helper functions for retireving Balancer pool data.

### SOR Object

When instantiating a new SOR object we must pass five parameters to the constructor:

`const SOR = new sor.SOR(Provider: JsonRpcProvider, GasPrice: BigNumber, MaxPools: number, ChainId: number, PoolsUrl: string)`

Where:

* Provider is an Ethereum network provider \(ex: local node or Infura\).
* GasPrice is used by the SOR as a factor to determine how many pools to swap against. i.e. higher cost means more costly to trade against lots of different pools. This value can be changed.
* MaxPools is the max number of pools to split the trade across. Limit to a reasonable number given gas costs.
* ChainId is the network chain ID \(i.e. 1=mainnet, 42=Kovan\)
* PoolsUrl is a URL used to retrieve a JSON list of Balancer Pools to be considered. Balancer labs currently keeps an updated list at:
  * Mainnet: [https://ipfs.fleek.co/ipns/balancer-bucket.storage.fleek.co/balancer-exchange/pools](https://ipfs.fleek.co/ipns/balancer-bucket.storage.fleek.co/balancer-exchange/pools)​
  * Kovan: [https://ipfs.fleek.co/ipns/balancer-bucket.storage.fleek.co/balancer-exchange-kovan/pools](https://ipfs.fleek.co/ipns/balancer-bucket.storage.fleek.co/balancer-exchange-kovan/pools)
  * Due to lots of IPNS caching issues the static storage can be used instead: [https://storageapi.fleek.co/balancer-bucket/balancer-exchange/pools](https://storageapi.fleek.co/balancer-bucket/balancer-exchange/pools)

### Fetching Pool Data

The SOR requires an up to date list of pool data when calculating swap information and retrieves on-chain token balances for each pool. There are two available methods:

#### `await SOR.fetchPools()` <a id="await-sor-fetchpools"></a>

This will fetch all pools \(using the URL in constructor\) and on-chain balances. Returns `true` on success or `false` if there has been an error.

#### `await SOR.fetchFilteredPairPools(TokenIn, TokenOut)` <a id="await-sor-fetchfilteredpairpools-tokenin-tokenout"></a>

A subset of valid pools for token pair, TokenIn/TokenOut, is found and on-chain balances retrieved. Returns `true` on success or `false` if there has been an error. This can be a quicker alternative to using fetchPools but will need to be called for every token pair of interest.

### Processing Swaps <a id="processing-swaps"></a>

#### `async SOR.getSwaps(...)` <a id="async-sor-getswaps"></a>

The `getSwaps` function will use the pool data and the trade parameters to perform an optimization for the best price execution. It returns swap information and the total that can then be used to execute the swaps on-chain.

```javascript
[swaps, total] = await SOR.getSwaps(
        tokenIn,
        tokenOut,
        swapType,
        swapAmount
    );
```

`tokenIn` - string: address of token in

`tokenOut` - string: address of token out

`swapType` - string: either `swapExactIn` or `swapExactOut`

`swapAmount` - BigNumber: amount to be traded, in Wei

#### `async SOR.setCostOutputToken(tokenOut)` <a id="async-sor-setcostoutputtoken-tokenout"></a>

The cost of the output token in ETH multiplied by the gas cost to perform the swap. This is used to determine whether the lower price obtained through including an additional pool in the transaction outweigh the gas costs. This function can be called before getSwaps to retrieve and cache the cost which will then be used in any getSwap calls using that token. Defaults to 0 for a token if not previously set.

Notice that tokenOut is tokenOut if swapType == 'swapExactIn' and tokenIn if swapType == 'swapExactOut.

### Example - Using SOR To Get List Of Swaps

Below is an example snippet that uses the SOR to return a final list of swaps and the expected output. The `swaps` returned can then be passed on to the exchange proxy or otherwise used to atomically execute the trades.

```javascript
require('dotenv').config();
import { SOR } from '@balancer-labs/sor';
import { BigNumber } from 'bignumber.js';
import { JsonRpcProvider } from '@ethersproject/providers';

// MAINNET
const tokenIn = '0x6B175474E89094C44Da98b954EedeAC495271d0F'; // DAI
const tokenOut = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'; // WETH


(async function() {
    const provider = new JsonRpcProvider(
        `https://mainnet.infura.io/v3/${process.env.INFURA}`
    );

    const poolsUrl = `https://ipfs.fleek.co/ipns/balancer-team-bucket.storage.fleek.co/balancer-exchange/pools`;

    const gasPrice = new BigNumber('30000000000');

    const maxNoPools = 4;

    const chainId = 1;

    const sor = new SOR(provider, gasPrice, maxNoPools, chainId, poolsUrl);

    // isFetched will be true on success
    let isFetched = await sor.fetchPools();

    await sor.setCostOutputToken(tokenOut);

    const swapType = 'swapExactIn';

    const amountIn = new BigNumber('1000000000000000000');

    let [swaps, amountOut] = await sor.getSwaps(
        tokenIn,
        tokenOut,
        swapType,
        amountIn
    );
    console.log(`Total Return: ${amountOut.toString()}`);
    console.log(`Swaps: `);
    console.log(swaps);
})()
```

### Example - SOR & ExchangeProxy

Balancer labs makes use of a [ExchangeProxy contract](https://github.com/balancer-labs/balancer-registry) that allows users to batch execute swaps recommended by the SOR. The following example shows how SOR and ExchangeProxy can be used together to execute on-chain trades.

```javascript
require('dotenv').config();
import { SOR } from '@balancer-labs/sor';
import { BigNumber } from 'bignumber.js';
import { JsonRpcProvider } from '@ethersproject/providers';
import { Wallet } from '@ethersproject/wallet';
import { MaxUint256 } from '@ethersproject/constants';
import { Contract } from '@ethersproject/contracts';

async function makeSwap() {
    // If running this example make sure you have a .env file saved in root DIR with INFURA=your_key, KEY=pk_of_wallet_to_swap_with
    const isMainnet = true;

    let provider, WETH, USDC, DAI, chainId, poolsUrl, proxyAddr;

    const ETH = '0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE';
    // gasPrice is used by SOR as a factor to determine how many pools to swap against.
    // i.e. higher cost means more costly to trade against lots of different pools.
    // Can be changed in future using SOR.gasPrice = newPrice
    const gasPrice = new BigNumber('25000000000');
    // This determines the max no of pools the SOR will use to swap.
    const maxNoPools = 4;
    const MAX_UINT = MaxUint256;

    if (isMainnet) {
        // Will use mainnet addresses - BE CAREFUL, SWAP WILL USE REAL FUNDS
        provider = new JsonRpcProvider(
            `https://mainnet.infura.io/v3/${process.env.INFURA}`
        );
        WETH = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'; // Mainnet WETH
        USDC = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48'; // Mainnet USDC
        DAI = '0x6B175474E89094C44Da98b954EedeAC495271d0F';
        chainId = 1;
        poolsUrl = `https://ipfs.fleek.co/ipns/balancer-team-bucket.storage.fleek.co/balancer-exchange/pools`;
        proxyAddr = '0x3E66B66Fd1d0b02fDa6C811Da9E0547970DB2f21'; // Mainnet proxy
    } else {
        // Will use Kovan addresses
        provider = new JsonRpcProvider(
            `https://kovan.infura.io/v3/${process.env.INFURA}`
        );
        WETH = '0xd0A1E359811322d97991E03f863a0C30C2cF029C'; // Kovan WETH
        USDC = '0x2F375e94FC336Cdec2Dc0cCB5277FE59CBf1cAe5'; // Kovan USDC
        DAI = '0x1528F3FCc26d13F7079325Fb78D9442607781c8C'; // Kovan DAI
        chainId = 42;
        poolsUrl = `https://ipfs.fleek.co/ipns/balancer-team-bucket.storage.fleek.co/balancer-exchange-kovan/pools`;
        proxyAddr = '0x4e67bf5bD28Dd4b570FBAFe11D0633eCbA2754Ec'; // Kovan proxy
    }

    const sor = new SOR(provider, gasPrice, maxNoPools, chainId, poolsUrl);

    // This fetches all pools list from URL in constructor then onChain balances using Multicall
    console.log('Fetching pools...');
    await sor.fetchPools();
    console.log('Pools fetched, get swap info...');

    let tokenIn = WETH;
    let tokenOut = USDC;
    let swapType = 'swapExactIn';
    let amountIn = new BigNumber('1e16');
    // This calculates the cost to make a swap which is used as an input to sor to allow it to make gas efficient recommendations.
    // Can be set once and will be used for further swap calculations.
    // Defaults to 0 if not called or can be set manually using: await sor.setCostOutputToken(tokenOut, manualPriceBn)
    await sor.setCostOutputToken(tokenOut);

    let [swaps, amountOut] = await sor.getSwaps(
        tokenIn,
        tokenOut,
        swapType,
        amountIn
    );

    console.log(`Total Expected Out Of Token: ${amountOut.toString()}`);

    console.log('Exectuting Swap Using Exchange Proxy...');

    const wallet = new Wallet(process.env.KEY, provider);
    const proxyArtifact = require('./abi/ExchangeProxy.json');
    let proxyContract = new Contract(proxyAddr, proxyArtifact.abi, provider);
    proxyContract = proxyContract.connect(wallet);

    console.log(`Swapping using address: ${wallet.address}...`);
    /*
    This first swap is WETH>TOKEN.
    The ExchangeProxy can accept ETH in place of WETH and it will handle wrapping to Weth to make the swap.
    */

    let tx = await proxyContract.multihopBatchSwapExactIn(
        swaps,
        ETH, // Note TokenIn is ETH address and not WETH as we are sending ETH
        tokenOut,
        amountIn.toString(),
        amountOut.toString(), // This is the minimum amount out you will accept.
        {
            value: amountIn.toString(), // Here we send ETH in place of WETH
            gasPrice: gasPrice.toString(),
        }
    );
    console.log(`Tx Hash: ${tx.hash}`);
    await tx.wait();

    console.log('New Swap, ExactOut...');
    /*
    Now we swap TOKEN>TOKEN & use the swapExactOut swap type to set the exact amount out of tokenOut we want to receive.
    ExchangeProxy will pull required amount of tokenIn to make swap so tokenIn approval must be set correctly.
    */
    tokenIn = USDC;
    tokenOut = DAI;
    swapType = 'swapExactOut'; // New Swap Type.
    amountOut = new BigNumber(1e18); // This is the exact amount out of tokenOut we want to receive

    const tokenArtifact = require('./abi/ERC20.json');
    let tokenInContract = new Contract(tokenIn, tokenArtifact.abi, provider);
    tokenInContract = tokenInContract.connect(wallet);
    console.log('Approving proxy...');
    tx = await tokenInContract.approve(proxyAddr, MAX_UINT);
    await tx.wait();
    console.log('Approved.');

    await sor.setCostOutputToken(tokenOut);

    // We want to fetch pools again to make sure onchain balances are correct and we have most accurate swap info
    console.log('Update pool balances...');
    await sor.fetchPools();
    console.log('Pools fetched, get swap info...');

    [swaps, amountIn] = await sor.getSwaps(
        tokenIn,
        tokenOut,
        swapType,
        amountOut
    );

    console.log(`Required token input amount: ${amountIn.toString()}`);

    console.log('Exectuting Swap Using Exchange Proxy...');

    tx = await proxyContract.multihopBatchSwapExactOut(
        swaps,
        tokenIn,
        tokenOut,
        amountIn.toString(), // This is the max amount of tokenIn you will swap.
        {
            gasPrice: gasPrice.toString(),
        }
    );
    console.log(`Tx Hash: ${tx.hash}`);
    await tx.wait();
    console.log('Check Balances');
}

makeSwap();
```

