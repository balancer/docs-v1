# Testing on Kovan

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/testing-on-kovan)

## Testing on Kovan

### Overview

This guide will walk through the process of interacting with Balancer on Kovan directly with the smart contracts along with additional tooling such as the subgraph and SOR.

#### Setup

This guide will use `seth` - a tool built by Dapphub to interact directly with smart contracts. To install, run the below command. Note: the Dapp Tools suite installs Nix OS.

```text
curl https://dapp.tools/install | sh
```

If this doesn't work, you might need to install nix manually first:  
`curl -L https://nixos.org/nix/install | sh`

{% hint style="warning" %}
Nix is broken on MacOS Catalina. To fix, follow the steps at: [https://github.com/NixOS/nix/issues/2925\#issuecomment-539570232](https://github.com/NixOS/nix/issues/2925#issuecomment-539570232)
{% endhint %}

Follow the remaining steps at: [https://github.com/dapphub/dapptools/tree/master/src/seth\#example-sethrc-file](https://github.com/dapphub/dapptools/tree/master/src/seth#example-sethrc-file) in order to select the `SETH_CHAIN` and address / key.

Run the below lines in your terminal to setup environment variables that will be used later in the guide.

```text
export BFACTORY=0x8f7F78080219d4066A8036ccD30D588B416a40DB
export FAUCET=0xb48Cc42C45d262534e46d5965a9Ac496F1B7a830
export WETH=0xd0A1E359811322d97991E03f863a0C30C2cF029C
export DAI=0x1528F3FCc26d13F7079325Fb78D9442607781c8C
export MKR=0xef13C0c8abcaf5767160018d268f9697aE4f5375
export USDC=0x2F375e94FC336Cdec2Dc0cCB5277FE59CBf1cAe5
export REP=0x8c9e6c40d3402480ACE624730524fACC5482798c
export WBTC=0xe0C9275E44Ea80eF17579d33c55136b7DA269aEb
export BAT=0x1f1f156E0317167c11Aa412E3d1435ea29Dc3cCE
export SNX=0x86436BcE20258a6DcfE48C9512d4d49A30C4d8c4
export ANT=0x37f03a12241E9FD3658ad6777d289c3fb8512Bc9
export ZRX=0xccb0F4Cf5D3F97f4a55bb5f5cA321C3ED033f244
```

#### Acquire Test Funds

All pools use wrapped ETH. In order to get WETH, either use the faucet at [https://faucet.kovan.network](https://faucet.kovan.network/) and then `deposit()` into the WETH contract. Or the faucet below may have some available WETH.

For all other tokens, Balancer has deployed a set of test tokens and a faucet on Kovan. The guide below will use DAI & MKR, but feel free to use any of the available tokens. An user can call `drip` once per day per token.

| Token | Address | Drip Amount |
| :--- | :--- | :--- |
| WETH | [0xd0a1e359811322d97991e03f863a0c30c2cf029c](https://kovan.etherscan.io/address/0xd0a1e359811322d97991e03f863a0c30c2cf029c) | 0.25 WETH |
| DAI | [0x1528F3FCc26d13F7079325Fb78D9442607781c8C](https://kovan.etherscan.io/address/0x1528F3FCc26d13F7079325Fb78D9442607781c8C) | 100 DAI |
| MKR | [0xef13C0c8abcaf5767160018d268f9697aE4f5375](https://kovan.etherscan.io/address/0xef13C0c8abcaf5767160018d268f9697aE4f5375) | 0.5 MKR |
| USDC | [0x2F375e94FC336Cdec2Dc0cCB5277FE59CBf1cAe5](https://kovan.etherscan.io/address/0x2F375e94FC336Cdec2Dc0cCB5277FE59CBf1cAe5) | 100 USDC |
| REP | [0x8c9e6c40d3402480ACE624730524fACC5482798c](https://kovan.etherscan.io/address/0x8c9e6c40d3402480ACE624730524fACC5482798c) | 10 REP |
| WBTC | [0xe0C9275E44Ea80eF17579d33c55136b7DA269aEb](https://kovan.etherscan.io/address/0xe0C9275E44Ea80eF17579d33c55136b7DA269aEb) | 0.02 WBTC |
| BAT | [0x1f1f156E0317167c11Aa412E3d1435ea29Dc3cCE](https://kovan.etherscan.io/address/0x1f1f156E0317167c11Aa412E3d1435ea29Dc3cCE) | 500 BAT |
| SNX | [0x86436BcE20258a6DcfE48C9512d4d49A30C4d8c4](https://kovan.etherscan.io/address/0x86436BcE20258a6DcfE48C9512d4d49A30C4d8c4) | 85 SNX |
| ANT | [0x37f03a12241E9FD3658ad6777d289c3fb8512Bc9](https://kovan.etherscan.io/address/0x37f03a12241E9FD3658ad6777d289c3fb8512Bc9) | 200 ANT |
| ZRX | [0xccb0F4Cf5D3F97f4a55bb5f5cA321C3ED033f244](https://kovan.etherscan.io/address/0xccb0F4Cf5D3F97f4a55bb5f5cA321C3ED033f244) | 400 ZRX |

Call `drip` on the faucet specifying a token address:

```text
seth send $FAUCET "drip(address)" $DAI
```

Confirm that you received test tokens:

```text
seth --from-wei $(seth --to-dec $(seth call $DAI "balanceOf(address)" $ETH_FROM))
```

Repeat the above steps for any additional tokens you would like to interact with on the Balancer test deployment.

#### Pool Creation

Next we're going to create a new pool. This guide will create a 50% WETH, 25% DAI, 25% MKR pool. Feel free to use any of the tokens

{% hint style="warning" %}
Some tokens do not use the standard 18 decimals. Be aware of balance calculations and weights when using tokens with different decimals.

USDC - 6  
WBTC - 8  
Any Compound token - 8
{% endhint %}

```text
seth send --gas 5000000 $BFACTORY "newBPool()"
```

Make note of the created contract address and set the below `BPool` variable to the address returned. This can be found on etherscan by looking at the internal txs of the tx hash.

```text
export BPOOL=0x... (address returned from previous step)
```

```text
BPOOL=0x... (address returned from previous step)
```

Token approvals are needed for any tokens you plan on adding to the pool. Approvals per pool is only required when interacting with the contracts directly. The frontend interfaces will all use a proxy so there will only be 1 time approvals to interact with all pools in the Balancer ecosystem.

```text
amount=$(seth --to-uint256 $(seth --to-wei 1000000 ether))
seth send $WETH "approve(address,uint256)" $BPOOL $amount
seth send $DAI "approve(address,uint256)" $BPOOL $amount
seth send $MKR "approve(address,uint256)" $BPOOL $amount
```

After all approvals are confirmed, tokens are bound to a specific pool by calling \`\`bind\(address token, uint balance, uint denorm\)\`\`\`

`denorm` is the denormalized weight for the token being added. Weights are stored in this fashion to prevent unnecessary gas costs of recalculating weights every time a new token is added. In our example, since we want 50% WETH, 25% DAI, and 25% MKR we can use the following denormalized weights: 10, 5, 5. \(Note: the denorm weights are scaled to Wei; i.e., "1" = 10^18, to allow for non-integer weights.\)

Valid denorm values range from 1 to 50 - and the total sum must be &lt;= 50. In practice, to ensure the math always works, best practice is to use a smaller range, such as 2 - 40.

```text
# Bind 1 WETH with a denormalized weight of 10
amount=$(seth --to-uint256 $(seth --to-wei 1 ether))
weight=$(seth --to-uint256 $(seth --to-wei 10 ether))
seth send $BPOOL "bind(address, uint256, uint256)" $WETH $amount $weight
```

Since we also want to bind DAI and MKR, and we know the desired weights, we have to calculate a balance for each token such that the internal token prices calculated by the protocol - based only on the weights and balances - match the external market prices of the tokens. \(Otherwise, there would be immediate, unintended arbitrage opportunities as soon as you created the pool.\) This example will assume the following asset prices: ETH - $200, MKR - $400, DAI - $1.

Since we bound 1 WETH at a price of $200 and we want that to be 50% of the pool, we want to bind $100 of both DAI and MKR.

```text
# Bind 100 DAI with a denormalized weight of 5
amount=$(seth --to-uint256 $(seth --to-wei 100 ether))
weight=$(seth --to-uint256 $(seth --to-wei 5 ether))
seth send $BPOOL "bind(address, uint256, uint256)" $DAI $amount $weight

# Bind 0.25 MKR with a denormalized weight of 5
amount=$(seth --to-uint256 $(seth --to-wei 0.25 ether))
weight=$(seth --to-uint256 $(seth --to-wei 5 ether))
seth send $BPOOL "bind(address, uint256, uint256)" $MKR $amount $weight
```

Let's confirm that all the tokens were added by using some view functions. This will get a token's normalized weight, or percentage of the pool, to make sure our original math was correct.

```text
seth call $BPOOL "getNumTokens()"

seth --from-wei $(seth --to-dec $(seth call $BPOOL "getNormalizedWeight(address)" $WETH))

seth --from-wei $(seth --to-dec $(seth call $BPOOL "getNormalizedWeight(address)" $DAI))

seth --from-wei $(seth --to-dec $(seth call $BPOOL "getNormalizedWeight(address)" $MKR))
```

Now that we're finished adding tokens, we need to set a swap fee for our pool. Otherwise we wouldn't earn anything on our assets and be more exposed to impermanent loss. In this example, we will set a swap fee of 0.3%:

```text
fee=$(seth --to-uint256 $(seth --to-wei 0.003 ether))
seth send $BPOOL "setSwapFee(uint256)" $fee
```

We have a valid pool with 3 bound tokens and a swap fee of 0.3%. At this point, a decision has to be made whether to `finalize` the pool. A pool can either be private or shared. A private pool allows the owner to continually adjust tokens, balances, weights, and fees. But prevents anyone else from adding or removing liquidity to that pool - it wouldn't be fair if the owner changed weights if other users have contributed liquidity! A shared pool is created when the `finalize` function is called and is a one-way transition. This locks all of the tokens, balances, weights, and opens the ability for outside users to add and remove liquidity.

A private pool allows the owner to continually adjust tokens, balances, weights, and fees - but prevents anyone else from adding or removing liquidity to that pool. After all, it wouldn't be fair if the owner changed weights after other users contributed liquidity! A shared pool is created when the `finalize` function is called - and this is a one-way transition. This locks all of the tokens, balances, weights, and opens the pool to outside users to add and remove liquidity.

For our example, we want other users to be able to add liquidity, so let's finalize it. 100 Balancer Pool Tokens will be minted as part of the pool finalization regardless of the bound token balances. Balancer Pool Tokens, or BPTs, represent proportional ownership of a pools liquidity. Any future joins or exits are calculated based on the relative liquidity being added.

```text
seth send $BPOOL "finalize()"
```

All done, now anyone can add liquidity and swap against the assets in the pool!

Let's confirm that we received BPTs by calling `balanceOf` directly on the pool address \(since the pools themselves are ERC20 tokens\)

```text
seth --from-wei $(seth --to-dec $(seth call $BPOOL "balanceOf(address)" $ETH_FROM))
```

