# BAL Liquidity Mining

### BAL Distribution Proportional to Liquidity on Balancer <a id="353e"></a>

To make the token distribution as fair as possible, we distribute BAL tokens proportional to the amount of liquidity each address contributed relative to the total liquidity on Balancer. Since there is liquidity in several different tokens, we use the USD value of liquidity as the common measure.

In practice, every week Balancer Labs has to:

* Define the starting and ending block of the week. Both are chosen as the block with the closest timestamp to a fixed weekly time \(e.g. Sunday 1:00pm UTC\). Say for example the starting block for a given week is \#10,100,000 and the ending block is \#10,140,000.
* Define snapshot blocks: at every 256 blocks \(i.e. every ~60 minutes\) counting backwards from the ending block until the starting block. For the example above, the snapshot blocks would be \#10,140,000; \#10,139,744; \#10,139,488; and so on.
* For each snapshot block, and for each Balancer pool, get the USD price for the tokens in the pool from [CoinGecko](https://www.coingecko.com/api/documentations/v3#/contract/get_coins__id__contract__contract_address__market_chart_) and calculate the USD liquidity in each pool.
* Since liquidity in pools that have lower trading fees contribute more to the protocol usage than liquidity in pools with high fees, we propose to multiply the USD pool liquidity by a feeFactor that weighs down pools according to their fee \(in %\):

![](https://miro.medium.com/max/980/1*3v3dcbR5t2m1JslUwrM2Jw@2x.png)

This creates the following bell-shaped curve for feeFactor, which means for example that a pool with a 0.5% fee has a feeFactor of ~0.94 and a pool with a 1% fee has a feeFactor of ~0.78:

![](https://miro.medium.com/max/2724/1*07aShFlZ136DIIx8ltsWPA@2x.png)

* **UPDATED for week 2** \(starting June 8th 00:00 UTC\), multiply the pool liquidity by a `ratioFactor`. Since pools that are imbalanced contribute less with trading volume \(because they offer higher slippage\), the community approved a [proposal to add a ratioFactor](https://forum.balancer.finance/t/introduction-of-a-weight-ratio-factor-in-liquidity-mining/15). This way highly imbalanced pools \(like with 98/2 weights\) have a much lower weight for the final BAL distribution.
* **UPDATED for week 3** \(starting June 15th 00:00 UTC\), multiply the pool liquidity by a `wrapFactor`. Since pools that contain pairs of tokens that have a hard peg \(e.g. DAI and cDAI\) do not contribute much with trading volume \(because traders can wrap DAI for cDAI and vice-versa\), the community approved a [proposal to add a wrapFactor](https://forum.balancer.finance/t/wrapfactor-penalizing-pairs-of-equivalent-tokens-in-liquidity-mining/28/3). This way, liquidity in such pairs \(like cETH/ WETH\) has a 0.1 wrapFactor \(i.e. counts 10 times less than for other regular pairs\), reducing the amount of BAL received by their liquidity providers. Pairs belonging to any of the following groups have a 0.1 wrapFactor applied: `[WETH, aETH, cETH, iETH (iearn), iETH (Fulcrum), PETH], // ETH group  [DAI, aDAI, cDAI, dDAI, iDAI, yDAI, rDAI, CHAI] // DAI group  [USDC, aUSDC, cUSDC, dUSDC, iUSDC, yUSDC] // USDC group`
* **UPDATED for week 5** \(starting June 29th 00:00 UTC\), After liquidity of all tokens is adjusted as usual by the currently active factors \(ratio, wrap, fee\), a `capFactor` has [been proposed](https://forum.balancer.finance/t/capfactor-capping-eligible-liquidity-to-10m-per-token/56), which is calculated such that every capped token is **limited to a maximum of $10M in adjusted liquidity**. `capFactor` is then applied to the liquidity of each affected capped token, resulting in an adjusted liquidity for every pool containing those tokens.`ETH, DAI, USDC, WBTC, BAL` is the list of uncapped tokens. Please read the proposal linked above for further details and examples.
* Calculate the proportional adjusted and capped \(see `capFactor` above\) liquidity USD value that each liquidity provider has in the pool. The table below shows an example for a pool that has 100$ worth of liquidity \(already adjusted by all factors\):

![](https://miro.medium.com/max/1472/1*2EM2KXgvt48qVK8FKQRmcw@2x.png)

* Divide the weekly amount of BALs distributed by the number of snapshot blocks. Considering blocks lasting 15s, a week would have a total of 40,320 blocks \(=7\*24\*60\*60/15\). Of these, there would be 158 snapshot blocks \(=40,320/256\). Considering 145,000 BAL distributed per week, the number of BAL distributed per snapshot block would be approximately 918 \(=145,000/158\).
* For each snapshot block, calculate BAL tokens to be received by each address. This is done proportionally to the total liquidity each address has on all Balancer pools divided by the total protocol liquidity. The table below shows an example of final distribution for a snapshot block.

![](https://miro.medium.com/max/1492/1*MvfWrMI2PovCLJiwaQr6EQ@2x.png)

All the calculations described above depend exclusively on on-chain data and historical token prices openly accessible on CoinGecko. This whole calculation process is fully auditable via [an open source script](https://github.com/balancer-labs) and BAL distribution results will be posted to IPFS and referenced on Balancer’s [twitter account](https://twitter.com/BalancerLabs).

### Token Whitelist and Eligible Pools for Liquidity Mining <a id="84fc"></a>

**UPDATED for week 5** \(starting June 29th 00:00 UTC\): All tokens present in the whitelist created by the community \(see whitelist proposal\) are eligible for BAL liquidity mining. The [most up to date list](https://github.com/balancer-labs/pool-management/blob/master/src/deployed.json) is maintained on Balancer's github. All tokens listed under the `"mainnet"` list in the json file linked are eligible. This list will evolve over time with input from the community.

Only Balancer pools containing 2 or more whitelisted tokens will be eligible for BAL liquidity mining.

