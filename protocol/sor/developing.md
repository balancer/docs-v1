# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/sor/developing)

# Developing

WIP - still needs to be cleaned up

```javascript
require('dotenv').config()
const sor = require('@balancer-labs/sor');
const BigNumber = require('bignumber.js');
const ethers = require('ethers');

// KOVAN
let tokenIn = '0x1528F3FCc26d13F7079325Fb78D9442607781c8C' // DAI
let tokenOut = '0xd0A1E359811322d97991E03f863a0C30C2cF029C' // WETH

// MAINNET
// let tokenIn = '0x6B175474E89094C44Da98b954EedeAC495271d0F' // DAI
// let tokenOut = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2' // WETH

function scale(input, decimalPlaces) {
    const scalePow = new BigNumber(decimalPlaces.toString());
    const scaleMul = new BigNumber(10).pow(scalePow);
    return new BigNumber(input).times(scaleMul);
}

const example = async function() {
    let pools = await sor.getPoolsWithTokens(tokenIn, tokenOut);

    if (pools.pools.length === 0)
        throw Error('There are no pools with selected tokens');

    let poolData = [];
    pools.pools.forEach(p => {
        let tI = p.tokens.find(t => ethers.utils.getAddress(t.address) === ethers.utils.getAddress(tokenIn))
        let tO = p.tokens.find(t => ethers.utils.getAddress(t.address) === ethers.utils.getAddress(tokenOut))
        let obj = {
            id: p.id,
            decimalsIn: tI.decimals,
            decimalsOut: tO.decimals,
            balanceIn: scale(tI.balance, tI.decimals),
            balanceOut: scale(tO.balance, tO.decimals),
            weightIn: scale(new BigNumber(tI.denormWeight).div(p.totalWeight), 18),
            weightOut: scale(new BigNumber(tO.denormWeight).div(p.totalWeight), 18),
            swapFee: scale(p.swapFee, 18),
        };

        poolData.push(obj);
    });

    const sorSwaps = await sor.smartOrderRouter(
        poolData,
        'swapExactIn',
        new BigNumber('10000000000000000000'),
        new BigNumber('10'),
        0
    );

    console.log(sorSwaps)
}

example()
```

