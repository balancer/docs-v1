# Liquidity Bootstrapping FAQ

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/smart-pools/liquidity-bootstrapping-faq)

## Liquidity Bootstrapping FAQ

### How do I launch a Liquidity Bootstrapping Pool?

Decide on critical parameters, such as sale duration, starting and ending weights, and estimate the demand \(i.e., expected sale rate\), using the [LBP simulator](https://docs.google.com/spreadsheets/d/1t6VsMJF8lh4xuH_rfPNdT5DM3nY4orF9KFOj2HdMmuY/edit?usp=sharing) to adjust the settings until you are happy with the resulting price curve. \(Best practice is to copy/download it, then customize to your own use case.\)

Post on [\#token-requests](https://discord.gg/ARJWaeF) to request eligibility for governance token rewards. \(At least a week’s advance notice is recommended, and BAL rewards require an active CoinGecko price feed, or at least "preview" mode.\) You can submit your pool metadata on [this form](https://docs.google.com/forms/d/18uAtnWJk7eVXl1VTnKAX5NmMJUST9pvvF43E3YtjT4E/) \(e.g., token symbol and name, icons for the token and pool, descriptive text, a url for the sale page, etc.\)

If your pool is eligible for weekly BAL rewards, they will be distributed to your LPs automatically. See the [Liquidity Mining ](../../core-concepts/bal-liquidity-mining/)page for staking and other special cases. If you contribute significant long-term liquidity to the platform, you can apply to have smart contract deployment gas costs reimbursed from the Balancer Ecosystem fund [here](https://docs.google.com/forms/d/e/1FAIpQLSeLAowndhx7n_ANMME16ZyPkvZgeJx0gcmP8loxoF0ZNMW6BQ/viewform).

Here's the general process to deploy a Liquidity Bootstrapping Smart Pool, conduct the token sale, and recover the proceeds:

* When you've decided on parameters as described above, [create the smart pool](../../guides/creating-a-smart-pool.md).
* Call updateWeightsGradually to start the sale. \(Note that unless you pause the pool, it will be visible to the exchange and available for trading immediately, though the weights are fixed until you start the update.\)
* \(Optionally\) call pokeWeights periodically to cause the weights to change on schedule. This is a public function that anyone can call - including from the public Pool Management interface. You can advertise this, and encourage buyers to poke for you \(it can be fairly expensive\). Large buyers would do well to pokeWeights before buying.
* At the end of the sale, Remove Liquidity to recover the proceeds; i.e., the reserve balance, plus any unsold tokens. If you reserved the add/remove tokens right \(and disallowed public LPs\), you can simply call removeToken. Otherwise, you can remove liquidity directly - though you need to leave at least some dust behind, since there is a minimum balance of each token.

### How long should an LBP last?

This is a fully customizable parameter that is up to you, based on your objectives. However we can provide some general tips. It’s important to give your potential investors sufficient time to participate and for healthy price discovery to occur. We’ve seen this process work well over a span of 3 days, but as LBPs are a new innovation, we believe there are other lengths of time that could work as well or even better.

Considering the fast pace and unpredictability of events happening regularly in DeFi, making your LBP too short could allow obstacles that are unrelated to your project \(i.e., a spike in Ethereum network congestion or a new yield farming craze\) to get in the way of a successful sale. We would recommend at least 3 days, but you are certainly free to make it shorter if you believe it would better serve your particular case.

#### _Note that the default minimum duration is approximately 2 weeks \(and 2 hours for the add token time lock\); using a shorter period will require overriding that default when you create the pool. This mainly applies to those writing scripts; if you use the smart pool GUI, the defaults are very short \(both 10 blocks\)._

### Should I start the sale immediately after deploying the pool?

Most LBPs do this, but some choose delay the start of the sale. There are many reasons for this, including waiting for the whitelisting/price feed to be set up, and wanting more time for pre-sale marketing/community building.

Projects that choose this method typically reserve the right to "pause swapping," and invoke it immediately after creating the pool. Since smart pools are created with swapping enabled, pausing is a separate transaction, so it is possible to "front-run" the pause transaction and perform a swap before the pause takes effect. \(Don't laugh - it's happened!\)

You can't beat human nature. Even though it's irrational, and the LBP announcements all encourage people to wait, we have observed an initial price spike in nearly every LBP to date. So if you choose to do a "delayed" LBP, you will be vulnerable to this sort of front-running.

The only way to completely prevent this is to write code to deploy and pause in a single transaction. Otherwise, one way to discourage early buying is to reserve the right to change the swap fee, and initialize it to 10% \(the max\). That way anybody front-running your pause transaction will pay a 10% fee on top of the high price. After the pause transaction is mined, you can drop the fee to the desired sale rate.

Any time there is a price spike - even with sales that start right away - arbers can exploit it. So it's more of a "user education" issue than a technical front-running problem. The long-term solution is to educate LBP buyers that patience is a virtue.

### How should I choose a starting price?

You can think of the starting price of your LBP as the ceiling you’d want to set for the token sale. This may seem counterintuitive, but since LBPs work differently than other token sales, your starting price should be set much higher than what you believe is the fair price.

This does not mean you’re trying to sell the token above what it’s worth. Setting a high starting price allows the changing pool weights of your LBP to make their full impact, lowering the price progressively until a market equilibrium is reached. Unlike older token sale models such as bonding curves, LBP investors are disincentivized to buy early, and instead benefit from waiting for the price to decrease until it reaches a level they believe is fair.

For example, if you believe the fair price for your token is $2, you may want to consider an opening price that is up to $10.

### What are the recommended starting and ending weight configurations for an LBP?

The general idea is to start with the pool weights skewed towards your token and end with the pool weights skewed towards the reserve asset\(s\). This configuration is designed to make it so that the majority of your tokens end up being exchanged for the reserve asset\(s\) you have chosen.

Be sure to think all the way through your intended sale, since there are some technical subtleties. For instance, if your weights are very close to the min/max weight boundaries, `pokeWeights` could fail in some edge cases where the total would temporarily be exceeded. Unless you need the maximum 98%/2% \(49/1 denorm\) weights, it's best to use weights that sum to a lower total number \(e.g., 38/2\), and put the project token first in the list when you create the pool, so that the weight _decrease_ would be processed first. \(We may be updating the GUI to avoid this issue by lowering the "resolution" in these cases - the max for a pool with canChangeWeights enabled would then be 96%/4%.\)

### How can I use an LBP to conduct a token sale with minimal seed capital?

The lower you set the weight\(s\) of the reserve asset\(s\) in your LBP, the less upfront capital you would need to seed your LBP. In a two-token LBP, the most skewed ratio you can set is 2:98, meaning that the pool composition will be 2% your reserve asset, and 98% the project token.

However, since the value of your project’s tokens is proportional to the value of the reserve assets you provide for the sale, the amount of funds you can raise in the sale is limited by the total value of the reserve assets you provide to the LBP.

For example, with an LBP weight ratio of 2:98 \(reserve asset:project token\), if your upfront capital is 50K USDC, and your starting price is 10 USDC, the amount of tokens you can sell is \[\(50K / 0.02\) / 10\] = approx. 250K tokens.

In contrast, if your upfront capital is 1M USDC, you’d be able to sell around 5M tokens.

### How can I calculate different scenarios for the amount of tokens to sell, based on the amount of seed capital and pool weights?

You can use our [LBP simulator](https://docs.google.com/spreadsheets/d/1t6VsMJF8lh4xuH_rfPNdT5DM3nY4orF9KFOj2HdMmuY/edit?usp=sharing) to plug in your variables and see the projected results. \(Best practice is to copy it so you have your own version, or even download to Excel.\)

There's a lot going on there, but a good place to start is the "ad hoc" simulator at the top right. There, you can type in balances, weights, and the swap fee, and see what the initial price would be. Then you can type your starting values into the main interface on the top left, and experiment with different ending weights and sale rates to come up with a reasonable price curve. It will also display the total proceeds and leftover tokens.

You'll see right away that it is very sensitive to the sale rate. The price is derived from two values - balances and weights. And you only control one of them! Your weight "flipping" schedule determines how the weights will change over time, but \(unless you intervene with further issuance or buybacks\), balances only change through trades: mostly people buying your token with the reserve currency.

If people buy while the weights are constant - or someone places a huge order - the token price may go up sharply. This is how a token sale can get "stuck" on a platform like Uniswap, where the prices are set by balances only. This may be optimal for retail liquidity, but is much less so for a token sale.

In an LBP, buyers suffering sticker shock need only wait for the weights to adjust and the price to start dropping. \(If they are impatient, they can call the public `pokeWeights` function themselves to tell the contract to update weights to the proper values at the current block.\)

In practice, we have seen that the market is efficient. Traders do indeed keep the price within a fairly narrow band around the discovered market value. That is one reason we believe this is the fairest token distribution method yet discovered. And as a bonus, this price curve tends to maximize total proceeds of the sale.

Note that it may not always be possible to sell all available tokens in the desired price range with the capital available. In this case, you could either split up the sale into multiple tranches \(either re-using the same pool, or making new pools, using the proceeds of each to fund the next\), or alter the supply or weight function during the sale in various ways, as described below.

### How can I influence the token price while running an LBP?

Unless you deposit tokens yourself during the sale \(e.g, by adding additional liquidity through joinPool\), the only way to affect prices during an LBP is by setting the target pool weights. Changing the target pool weights influences price, such that increasing the weight of an asset increases its relative price, and vice versa. The change in price gradually incentivizes investors to buy your token in exchange for the reserve asset\(s\), which then changes the balances of assets in the pool.

The design of the LBP tends to keep the sale price fairly even \(or slightly declining\) during the sale. However, this depends on the market buying pressure, which can only be estimated until the sale actually begins. If you have a really great project, or strong marketing, it is possible for buyers to "overpower" the weight changes and make the price drift upwards.

Conversely, if there is too little buy pressure, the price might drop faster than expected. In either case, it is possible to make the "price curve" steeper or shallower by adjusting the end weights. You can do this \(as described below\), up until the end of the sale, limited by your "minimum duration." For instance, if you've set the minimum duration to 2 hours, you can adjust the weights \(or fine-tune the end block\) until 2 hours before the final block, without extending the sale.

### How do I change the weights of my LBP while it’s live?

Use the `updateWeightsGradually` function to put the contract into a state where it will respond to the `pokeWeights` call by setting all the weights according to the point on the "weight curve" corresponding to the current block. `pokeWeights` can be called by you, or anyone, to update the weights according to your LBP’s configuration.

You can also call `updateWeight` \(as long as a gradual update is not running\) to set weights arbitrarily - but this will not affect the price. It will transfer tokens as required such that the new balances and weights maintain current prices.

The GUI handles this for you, but in case you're **not** going through the GUI, and setting weights manually or programmatically, take care to use the same total denorm weights at the starting and ending points. For instance, to go from 96%/4% to 30%/70%, the start weights would be 24/1, and the end weights would be 7.5/17.5 - notice that both total to 25. The contract always interpolates linearly between the start and end weights - but the resulting price curve will only be linear if the weight totals match.

For instance, if you started at 24/1, and ended at 15/35, the percentages work out the same - but the weight curve would be shallower \(i.e., go down slower\).

### What are the reserve assets I can sell my token for?

You can sell your token for any ERC-20 tokens. You can choose up to 7 other tokens to be used as reserve assets in your LBP token sale. Projects typically sell their tokens for highly liquid stablecoins such as DAI or USDC; and/or for WETH.

### How do I get my token listed by name and logo on the Balancer exchange UI?

The Balancer team regularly monitors the crypto landscape and adds new tokens to our listings based on internal requirements. Tokens do not need to request to be listed on the exchange, as it is done proactively.

If you are launching an LBP and want to make sure that your token is listed on the exchange before launch, please [contact the team](mailto:contact@balancer.finance) for assistance.

### How do I get my token whitelisted for BAL mining earnings?

[This page](../../core-concepts/bal-liquidity-mining/exchange-and-reward-listing.md) describes the process.

### After providing the initial seed capital needed to launch an LBP, do I need to deposit additional capital later?

No, you do not need any additional capital beyond the initial seed amount based on the starting weights you’ve selected for your pool. However, you do have the optional ability to deposit new capital into the pool as a buyback mechanism while the LBP is running.

If you use the LP whitelisting to disable public LPs, and wish to add liquidity yourself later, you will need to add your DSProxy address \(if you're using the GUI\) to the whitelist in order to do so. \(On the smart pool GUI, this address can be found on the About page, under "smart pool controller".\)

### Can I deploy/control the LBP from a script?

Yes. Conversely, if you wrote your own script to deploy the pool \(vs using the smart pool GUI\), you can also manage it from the GUI. To do this, you would call setController \(or use that feature on the Settings page, if you're starting from the GUI\), and set it to the address you want to control the pool. You could deploy through the GUI, then `setController` to the account running the script \(e.g., to set the swap fee dynamically based on an off-chain algorithm\). To transfer it back, just call `setController` again from that script, setting it to your DSProxy address.

DSProxy contracts are "helper contracts" deployed per user account to make interacting with multiple pools easier and more gas-efficient. If you're starting from a script and don't have one, you can start to create a pool through the GUI, and it will prompt you to create a proxy as the first step.

The[ Pool Management GUI](https://github.com/balancer-labs/pool-management-vue) and [Configurable Rights Pool](https://github.com/balancer-labs/configurable-rights-pool) are both open source. You can refer to the large test suite for many examples of how to interact with the CRP, and both the CRP suite and the Vue app contain helper functions for things like slippage and adding/removing liquidity. There are also many simulators available, linked at the bottom of the [CRP Tutorial](../../guides/crp-tutorial/).

We've tried to make it as clear and straightforward as possible - but there are subtleties the GUI handles that you would need to hand-code in a script. For instance, balances for tokens with less than 18 must be "normalized." This includes many common tokens; e.g., USDC has 6, Compound cTokens all have 8, etc.

For instance, a balance of "10" in wei would be "10000000000000000000" for DAI \(with 18 decimals\), but "10000000" for USDC \(with 6 decimals\).

One final note - if you deploy a CRP through a script, we recommend using the standard `CRPFactory` \(addresses [here](../addresses.md)\). If you deploy it "directly", it will still work, but will not be recognized by the BAL mining scripts, and you will need to do a [redirect](../../core-concepts/bal-liquidity-mining/).

### Can I use a "Pausable" token in the LBP?

You can, but there are special considerations in this case! Since Balancer is a permissionless protocol, once your tokens are "in the wild," anyone can do anything they want with them - including creating new Balancer pools \(or Uniswap pools, or pools on any other protocol\).

Normally this doesn't hurt anything - but if you pause the token contract after the sale - preventing all transfers - anyone who added liquidity to any of these other pools will no longer be able to withdraw their funds.

For this reason, your LBP pool definitely needs to use the "Must whitelist LPs" right to prevent public LPs - otherwise they could get "stuck" in this manner.

Since there is no way to prevent token holders from creating new Uniswap, Balancer, etc. pools during the sale, it would be a good idea to mention in your announcement that there is only **one** official pool - which does **not** allow adding liquidity - and any users who add liquidity to any "imposter" pools, or send them to any account they do not control, will lose funds if they fail to withdraw them before the end of the sale.

### Can I allow public Liquidity Providers?

You can \(though see above for special considerations when using non-standard token contracts\). If you do not reserve the "Must whitelist LPs" right, anyone will be able to add liquidity to your pool. It is also possible to use the "cap" right to limit the total BPT supply \(and therefore total value of the pool\).

If you take this approach, there are a few things to keep in mind.

* You will not be able to end the sale with removeToken, since you will not have anough BPT, and will need to "Remove Liquidity" to terminate the sale. Note that you cannot remove 100% from a smart pool, since there are minimum balance requirements, but you can remove very close to 100%, down to dust - whatever leaves all balances &gt; 10^-6. One safe technique is to remove 99.9% first, see what's left, and then remove 99.9% again \(if it's worth the gas\).
* It is possible for "whales" to unbalance your pool by adding large amounts of "single-sided" liquidity, instantaneously changing the prices, possibly enough to induce arbitrage and impermanent loss. Large initial balances \(and swap fees\) mitigate this risk. Also, the protocol prevents adding more than half the current balance of any token in a single transaction.
* Since Balancer is a permissionless protocol, anyone can create new pools. Perhaps you have a staking protocol, and encourage LPs to stake their BPTs for additional earnings. In this case, it's important to make your users aware there might be "counterfeit" pools, and clearly direct them to **your** pool, whose tokens your staking contract accepts.

