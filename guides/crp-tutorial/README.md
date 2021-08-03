# CRP Tutorial

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/crp-tutorial/)

## CRP Tutorial

Similar to Core Pools, Configurable Rights Pools are created from a public factory. Refer to [Addresses](../../smart-contracts/addresses.md) to find the contract on your network of choice.

To deploy a new Configurable Rights Pool, call newCRP on the CRPFactory. This function takes three parameters \(cheating a bit, since two of them are big structs\):

```text
function newCrp(
    address factoryAddress,
    ConfigurableRightsPool.PoolParams calldata poolParams,
    RightsManager.Rights calldata rights
)
    external
    returns (ConfigurableRightsPool)
```

The factoryAddress is that of the BFactory \(see [Core Concepts](../../smart-contracts/smart-pools/concepts.md)\). The Configurable Rights Pool is the Smart Pool "wrapper" around the underlying Core Pool \(BPool\). You \(caller of newCRP\) are the controller of the Smart Pool. The Smart Pool is the controller of the Core Pool. So the Smart Pool needs to deploy a Core Pool - and requires the BFactory to do that.

The next argument is PoolParams - this is where you define the structure and basic parameters of the pool, such as the tokens it will hold, their initial weights and balances, and the swap fee.

```text
struct PoolParams {
    // Balancer Pool Token (representing shares of the pool)
    string poolTokenSymbol;
    string poolTokenName;
    // Tokens inside the Pool
    address[] constituentTokens;
    uint[] tokenBalances;
    uint[] tokenWeights;
    uint swapFee;
}
```

Since the Balancer Pool Tokens are themselves ERC20 tokens, they have symbols and names. You can set both when creating your pool.

The tokens must be addresses of conforming ERC20 tokens. Balances and weights are expressed in Wei - and the weights are denormalized, not percentages. Valid denormalized weights range from 1 to 49, since the maximum total denormalized weight is 50. \(This corresponds to a percentage range from 2% to 98%: 1/\(1+49\) = 2%; 49/\(1+49\) = 98%\)

**Note that balances must be "normalized" for the number of decimals in the token. For instance, USDC has 6 decimals, so "10" is "10000000" - not "10000000000000000000"!**

{% hint style="warning" %}
Note that if you're going to be doing gradual weight updates, using denorm totals near the maximum 50 can be problematic! It is possible for `pokeWeights`to fail if the weights are 45/5, since the contract will never allow the total to go over 50 - even temporarily during an intermediate step. Unless you truly need the full range, best practice is to use a lower total, like 40; e.g.; for 90/10, use 36/4. \(If you need to use extreme weights, put tokens whose weights should go down first in the array; that way the weight reductions will be processed before the increases.\)
{% endhint %}

{% hint style="info" %}
Also note that adding new tokens \(to pools with the addRemoveTokens right\), and direct calls to updateWeight have stricter decimal limits. Core pools support tokens from 6-18 decimals. Smart pools can hold the full range of tokens - but only support addToken/updateWeight operations on 12-18 decimal tokens.
{% endhint %}

The swap fee is also expressed in Wei, as a percentage. For instance, toWei\("0.01"\) means a 1% fee.

Finally, the Rights struct defines the permissions.

```text
struct Rights {
    bool canPauseSwapping;
    bool canChangeSwapFee;
    bool canChangeWeights;
    bool canAddRemoveTokens;
    bool canWhitelistLPs;
    bool canChangeCap;
}
```

{% hint style="warning" %}
Note that if you are whitelisting LPs, and intend for the whitelisted users to Add Liquidity through the Balancer GUI, you will need to whitelist their DSProxy addresses - not their wallet address. \(The DSProxy address can be found at the bottom of the wallet page. New users will need to create a DSProxy before adding liquidity.\)
{% endhint %}

At this point \(after calling newCRP\), we have a deployed Configurable Rights Object with all its permissions and parameters defined. But we can't do much with it - mainly because there is no Core Pool yet. We need to deploy a new Core Pool, with our Smart Pool as the controller, by calling createPool\(initialSupply\). \(There is also an overloaded version of createPool; more on that later.\)

We've already defined the tokens and balances we want the pool to hold. When we call createPool with a value for initialSupply, it will mint _initialSupply_ Balancer Pool Tokens \(BPTs\) and transfer them to the caller, simultaneously pulling the correct amount of collateral tokens into the contract. \(They end up in the Core Pool, passed through the CRP.\)

To accomplish this, we need to allow the CRP to spend our collateral tokens, before calling createPool. For an example three-token pool, we might write:

```text
const MAX = web3.utils.toTwosComplement(-1);

// crpPool was returned from CRPFactory.newCRP()

await weth.approve(crpPool.address, MAX);
await dai.approve(crpPool.address, MAX);
await xyz.approve(crpPool.address, MAX);

// consume the collateral; mint and xfer 100 BPTs to caller
await crpPool.createPool(toWei('100'));
```

Now we're in business! The pool will already be set up for public swapping, and depending on the permissions settings, possibly public liquidity provision as well. The Core Pool will show up on the Exchange GUI.

{% hint style="info" %}
Refer to [Exchange and Reward Listing](../../core-concepts/bal-liquidity-mining/exchange-and-reward-listing.md) for instructions on adding any new tokens you might be introducing to the Exchange GUI, and making them eligible for BAL governance tokens. Earnings are distributed weekly, and are not applied retroactively, so you'll need to have the token approved by 00:00 UTC the Monday **before** you launch your pool.
{% endhint %}

{% hint style="danger" %}
If your smart pool is eligible for BAL, earnings will be redirected to LPs - as long as you create the pool through our standard factory. If you create a new pool using a different factory, or deploy a pool contract directly, you will need to apply for a redirect or redistribution. \(You will also need a redirect if your CRP controller is a contract that holds BPTs, and doesn't have a way to withdraw them.\)

The process for the redirect is to make a pull request to update [this file](https://github.com/balancer-labs/bal-mining-scripts/blob/master/config/redirect.json) in our script repository with the CRP and your wallet address, along with proof that you own the pool \(e.g., the CRP deployment transaction hash\). Here's an [example request](https://github.com/balancer-labs/bal-mining-scripts/pull/11). Similarly, if you have a CRP and want to handle the redistribution differently, you can make a pull request to update [this file](https://github.com/balancer-labs/bal-mining-scripts/blob/master/config/redistribute.json).

Here is a [diagram](https://drive.google.com/file/d/13QOMv-PVNZqJwdz9g6QKFqBUJwOypkZY/view) detailing how we compute BAL mining earnings.
{% endhint %}

{% hint style="warning" %}
On a related note, while it is possible to send tokens directly to a core pool contract - as long as they are in the pool, you they can be recovered with gulp\(\) - this is not the case for smart pools! Any tokens sent directly to the CRP contract will be unrecoverable. \(This can happen in some circumstances even without a direct transfer, such as airdrops to token holders.\)
{% endhint %}

Note that there is also an overloaded version of createPool, where you can specify additional parameters related to updateWeightsGradually.

```text
function createPool(
    uint initialSupply,
    uint minimumWeightChangeBlockPeriod,
    uint addTokenTimeLockInBlocks
)
```

If the pool you're creating doesn't have permission to change weights, or you accept the default values of the time parameters, you can just use the single-argument version.

_initialSupply_ can be set to a value of your choice - within min/max limits \(currently 100 - 1 billion, in ether units\).

_minimumWeightChangeBlockPeriod_ enforces a minimum time between the start and end blocks of a gradual update. _addTokenTimeLockInBlocks_ is used when adding a token \(if you have the AddRemoveTokens permission\), as the minimum wait time between committing a new token, and applying it \(i.e., actually adding it to the pool\). There is one additional constraint: _minimumWeightChangeBlockPeriod &gt;= addTokenTimeLockInBlocks._

These parameters have default values, and can only be changed by overriding them in this version of createPool. The addTokenTimeLock defaults to 500 blocks \(~ 2 hours\), and the blockPeriod defaults to 90,000 \(~ 2 weeks\). _**If you want to use different minimum values, be sure to set them in createPool, since they are immutable thereafter!**_

Also note that these only apply to gradual updates. The controller can call updateWeight directly at any time, as long as no gradual update is in progress.

{% hint style="warning" %}
The GUI displays the approximate dates for the block numbers you enter. Some protocols \(e.g., UMA\) use UNIX timestamps for time-related inputs. If you are calling it programmatically, make sure you use _block numbers_, not timestamps!

Luckily, `updateWeightsGradually` is forgiving - even if this happens, you don't have to wait five centuries for the block numbers to catch up to the timestamp \(currently ~ 2600 AD\); just call it again with the correct values.
{% endhint %}

You can use [this simulator](https://docs.google.com/spreadsheets/d/1t6VsMJF8lh4xuH_rfPNdT5DM3nY4orF9KFOj2HdMmuY/edit#gid=1392289526) to explore how weight changes and updates work, estimate slippage and impermanent loss, etc. There is also a [demo video](https://vimeo.com/466075719) illustrating the simulator functionality.

Complex functionality = lots of simulators! [This one](https://docs.google.com/spreadsheets/d/1eELxUXdPZ2srDs82xI1paod6_IvgNps-hf4hvUvHDWM/edit#gid=1103637370) helps you calculate single- and multi-asset entry and exit.

