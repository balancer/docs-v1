# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/limitations)

# Limitations

Balancer is meant to be a flexible and agnostic DeFi primitive. Due to constraints such as gas and math approximations, there are some limitations built into the protocol.

**ERC20 Tokens**

ERC20 compliance: pool tokens have to be ERC20 compliant. Bronze does not support ERC20 tokens that do not return `bools` for `transfer` and `transferFrom`. ****There are no upgrade mechanisms in the contracts to allow for token upgrades. Any upgrade will need to be manually coordinated and moved into new pools.

Tokens that have internal transfer fees or other non-standard balance updates may create arbitrage opportunities. Ex: DGX has both a demurrage and a transfer fee that will change a pool's balance compared to the internal accounting balance

**Minimum Bound Tokens - 2**

A functional pool must contain at least two tokens. \(If the pool creator can remove tokens, it is possible to remove them all and have one or zero, but of course no swaps are possible if there is only one token, and a 0-token pool cannot be restored.\)

**Maximum Bound Tokens - 8**

The maximum number of tokens that can be in a given pool is 8.

**Maximum Swap In Ratio - 1/2**

A maximum swap in ratio of 0.50 means a user can only swap in less than 50% of the current balance of `tokenIn` for a given pool

**Maximum Swap Out Ratio - 1/3**

A maximum swap out ratio of 1/3 means a user can only swap out less than 33.33% of the current balance of `tokenOut` for a given pool

**Minimum Swap Fee - 0.0001%**

There is a minimum swap fee of 0.0001% \(or a hundredth of a basis point\) to counteract any unfavorable pool rounding.

**Maximum Swap Fee - 10%**

This is to prevent malicious pool controllers from setting predatory trading fees. \(For instance, a pool controller could front-run a large trade and set the fee to 99%.\) No one wants to be [this guy](https://etherchain.org/tx/c215b9356db58ce05412439f49a842f8a3abe6c1792ff8f2c3ee425c3501023c).

**Minimum Balance - \(10^18\) / \(10^12\)**

The minimum balance of any token in a pool is 10^6 wei. **Important**: this is agnostic to token decimals and may cause issues for tokens with less than 6 decimals. Also note that this is only enforced on initial token binding. Future exits can potentially bring the pool below the minimum balance threshold and users should be aware of potential rounding errors.

**Min/Max Initial BPT Supply - \(100 / 1 Billion\)**

Core Balancer Pools have a fixed initial token supply of 100 \(i.e., BPTs that represent shares of the pool's liquidity\). Smart Pools allow the pool creator to specify an initial supply within these bounds.



