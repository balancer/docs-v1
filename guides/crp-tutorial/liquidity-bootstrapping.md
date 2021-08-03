# Liquidity Bootstrapping Example

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/crp-tutorial/liquidity-bootstrapping)

## Liquidity Bootstrapping Example

Let's walk through a complete example, using the [Liquidity Bootstrapping](https://balancer.finance/2020/03/04/building-liquidity-into-token-distribution/) use case. There is also extensive higher-level discussion in the [Liquidity Bootstrapping FAQ](../../smart-contracts/smart-pools/liquidity-bootstrapping-faq.md).

First, we give the token a symbol and name, set the basic pool parameters, and determine the permissions. All we really need to be able to do is change the weights, so we can set all the other permissions false.

As noted earlier, setting the permissions as strict as possible minimizes the trust investors need to place in the pool creator. Liquidity providers for this pool can rest assured that the fee can never be changed, no tokens can be added or removed, and they cannot be prevented from adding liquidity \(e.g., by being removed from the whitelist, or having the cap lowered\).

**Note that balances must be "normalized" for the number of decimals in the token. For instance, USDC has 6 decimals, so "10" is "10000000" - not "10000000000000000000"!**

```text
// XYZ and DAI are addresses
// XYZ is the "project token" we're launching

const poolParams = {
    tokenSymbol: 'LBT',
    tokenName: 'Liquidity Bootstrapping Token',
    tokens: [XYZ, DAI],
    startBalances: [toWei('4000'), toWei('1000')],
    startWeights: [toWei('32'), toWei('8')],
    swapFee: toWei('0.005'), // 0.5%
}

const permissions = {
    canPauseSwapping: false,
    canChangeSwapFee: false,
    canChangeWeights: true,
    canAddRemoveTokens: false,
    canWhitelistLPs: false,
    canChangeCap: false
};
```

Next, we use these structs to deploy a new Configurable Rights Pool \(and the underlying Core Pool\).

```text
// If deploying locally; otherwise use the published addresses
// This is the factory for the underlying Core BPool
bFactory = await BFactory.deployed();

// This is the Smart Pool factory
crpFactory = await CRPFactory.deployed();

// Static call to get the return value
const crpContract = await crpFactory.newCrp.call(
    bFactory.address,
    poolParams,
    permissions,
);

// Transaction that actually deploys the CRP
await crpFactory.newCrp(
    bFactory.address,
    poolParams,
    permissions,
);

// Wait for it to get mined
const crp = await ConfigurableRightsPool.at(crpContract);

// Creating the pool transfers collateral tokens
// Must allow the contract to spend them
await dai.approve(crp.address, MAX);
await xyz.approve(crp.address, MAX);

// Create the underlying pool
// Mint 1,000 LBT pool tokens; pull collateral into BPool
// Override with fast block wait times for testing purposes
//   (Defaults are 2 hours min delay / 2 weeks min duration)
await crp.createPool(toWei('1000'), 10, 10);
```

At this point we have an initialized pool. The admin account has 1,000 LPTs, and the underlying BPool is holding the tokens. The pool is enabled for public trading and adding liquidity.

To facilitate the token launch - with low slippage, low initial capital, and stable prices over time, per the paper referenced above - we want to gradually "flip" the weights over time. We start with the project token at a high weight \(32/\(32+8\), or 80%, and collateral DAI at 20%. At the end of the launch, we want XYZ at 20%, and DAI at 80%. We accomplish this by calling updateWeightsGradually; we're allowed to do this because the canChangeWeights permission was set to true.

```text
// Start changing the weights in 100 blocks
const block = await web3.eth.getBlock('latest');
const startBlock = block.number + 100;
const blockRange = 10000;

// "Flip" the weights linearly, over 10,000 blocks
const endBlock = startBlock + blockRange;

// Set the endWeights to the reverse of the startWeights
const endWeights = [toWei('8'), toWei('32')];

// Kick off the weight curve
await crp.updateWeightsGradually(endWeights, startBlock, endBlock);
```

Of course, smart contracts can't change state by themselves. What _updateWeightsGradually_ actually does is put the contract into a state where it will respond to the _pokeWeights_ call by setting all the weights according to the point on the "weight curve" corresponding to the current block. \(It will revert before the start block, and set the weights to their final values if called after the end block.\)

```text
// Sample code to print the current weights,
// Then call pokeWeights to update them

for (i = 0; i < blockRange; i++) {
    weightXYZ = await controller.getDenormalizedWeight(XYZ);
    weightDAI = await controller.getDenormalizedWeight(DAI);
    block = await web3.eth.getBlock("latest");
    console.log('Block: ' + block.number + '. Weights -> XYZ: ' +
        (fromWei(weightXYZ)*2.5).toFixed(4) + '%\tDAI: ' +
        (fromWei(weightDAI)*2.5).toFixed(4) + '%');

    // Actually cause the weights to change
    await crp.pokeWeights();
}
```

There are many subtleties; for instance, you could implement a non-linear bootstrapping weight curve by calculating the weights off-chain and setting them directly. For a comprehensive set of tests that demonstrate all features of the Configurable Rights Pool, see our [GitHub](https://github.com/balancer-labs/configurable-rights-pool).

