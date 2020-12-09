# Liquidity Bootstrapping Pool

Here again is the [article](https://balancer.finance/2020/03/04/building-liquidity-into-token-distribution/) describing this common case - basically a sustainable 2020 reboot of the 2017 "ICO" - minus the worst of the legal and regulatory issues. \(The latest term seems to be IDO - Initial DEX Offering.\)

The idea is to launch a token with low capital requirements, by setting up a two-token pool with a project and a collateral token. The weights are initially set heavily in favor of the project token, then gradually "flip" to favor the collateral coin by the end of the sale. The sale can be calibrated to keep the price more or less steady \(maximizing revenue\), or declining to a desired minimum \(e.g., the initial offering price\).

Legend**: required;** ~~not required;~~ _optional_

**Rights configuration:**

* _canPauseSwapping_
* ~~canChangeSwapFee~~
* **canChangeWeights**
* _canAddRemoveTokens_
* **canWhitelistLPs**
* ~~canChangeCap~~

Pausing swapping is an optional feature here. You might want to halt trading for various reasons \(e.g., stronger than expected demand is driving up the price, and people are selling tokens back to the pool for profit instead of buying them\). There isn't much of a trust issue here - the purpose is to sell tokens, so the pool operator has an incentive to leave swapping enabled.

In a token sale, you want to keep fees low to encourage trading; there's really no reason to change it. So best practice would be set it to the minimum on creation and don't enable changing it. You want to enable as few rights as possible. More rights = more possible manipulation from the pool operator = more trust required.

Since the core of the strategy is changing weights, you absolutely need to enable that right.

The add/remove tokens right depends on your future intentions, since there are two ways to remove your liquidity at the conclusion of the sale. If this is a one-off auction, you can enable the right, and redeem through two "removeToken" calls. This is simple, but destroys the pool.

If you want to re-use the pool \(e.g., there are "phases" or multiple releases\), you can leave this right disabled, and retrieve the proceeds through repeated calls to exitPool. You cannot remove 100% of the liquidity through exitPool, since token balances cannot go to zero, and single asset exit is limited to 1/3 of the total balance per transaction. To add more project tokens, you would need to add yourself to the whitelist, and joinPool.

In a token sale \(vs an investment pool\), you don't want anyone else providing liquidity. Only the pool creator should be issued pool tokens, and only they should be able to redeem the proceeds at the end. This is most easily accomplished by using the whitelist. If you enable this right and don't add anyone to the whitelist, no one can "join" the pool.

You could also prevent others from adding liquidity through the cap right. If you enable it, the cap will be set to the initial supply, which has the same effect as the whitelist. However, you will need to be careful when you redeem the pool tokens at the end of the sale, since any token redemption would reduce the supply below the cap and present a window for others to join the pool. To prevent this, you could set the cap to 0 before withdrawing the proceeds.

Watch out for some subtleties here! Some LBP owners might want to allow public LPs; Uniswap does, after all. If you don't have the whitelist right or the cap right, anyone can add liquidity. This means the LBP controller is not the sole "owner" of the pool, since others also have pool tokens. Consequently, if the controller tries to destroy the pool after the sale with calls to removeToken -- it won't work, because they won't have enough pool tokens. This makes sense - otherwise, pool owners could "drain the pool" just by calling `removeToken`!

Depending on the balances, removing one of the tokens "might" work. For instance, say you're at the end of the sale, so you have a small balance of your project token, and a large balance of DAI. You would probably have enough pool tokens to remove the project token. At that point, you would have a 1-token pool of DAI, and -- unless all the other LPs exit the pool - you would not be able to call `removeToken` on DAI. You could only exitPool with whatever pool tokens you had - and the remaining DAI balance would represent the amount of liquidity added by the other LPs.

So the upshot of all this is if you intend to allow public LPs on your LBP, maybe you don't need the add/remove token right. \(This would make your pool more trustless.\)

It could also happen that you don't have enough to call `removeToken` on either. For instance, if a purchaser sold a large amount of your project token back into the pool, or someone added a large amount of liquidity. If you need to call `removeToken` to reuse the pool, remember you can always "buy out" the other LPs. If you're 250 pool tokens short of what you need to remove the project token, you can join with 250 pool tokens' worth of DAI, after which `removeToken` will succeed. All the other LPs wishing to withdraw their liquidity would get their proceeds entirely in DAI.

This is something to keep in mind when investing in Balancer Pools in general - when you withdraw liquidity, you will get the token\(s\) in whatever proportion they are in at the time of withdrawal. If the pool controller has the add/remove token right, they might not even be the same tokens. \(This would surely be the case for "perpetual synthetic" pools that add and remove synthetics as they are minted and expire.\)

One other important consideration is the "block time" settings - the minimum time between weight updates, and the minimum duration of each gradual update. These times default to 2 hours and 2 weeks, meaning the weights cannot change faster than every 2 hours, and once you start a gradual update, you cannot add a token or do a manual update for 2 weeks.

You can make these periods arbitrarily short \(even zero\), but then the pool operators have greater control, and more trust is required.

Also keep in mind that contracts cannot change state by themselves - weights only change when someone calls `pokeWeights`. A common mechanism is for the pool operator to set up something like a cron job to call it automatically, though it is a public function that anyone can call.

Note that `pokeWeights` changes weights, but not balances - therefore, it can change the price \(albeit only along the trajectory set by the last call the updateWeightsGradually\).

The controller function `updateWeight` changes weights directly - but not the price. This means it must also adjust the balances at the same time \(i.e., transfer tokens\). So the controller can set weights arbitrarily -- but not while an `updateWeightsGradually` is running. And they must have the tokens available to do it.

Though most LBPs are two-token pools \(e.g., a project token and a single reserve currency, usually a stable coin\), it is possible to have three tokens, or more. For instance, a stable coin and WETH, in addition to the project token. [One project](https://pools.balancer.exchange/#/pool/0x10996ec4f3e7a1b314ebd966fa8b1ad0fe0f8307/) was recently launched that way. There is a [simulator](https://docs.google.com/spreadsheets/d/10434m342Rt0rTarCd2SCtMizT9GrMHrK/edit#gid=1694604160) available for this case.





