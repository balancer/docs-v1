# Limitations

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/core-concepts/protocol/limitations)

## Limitations

Balancer is meant to be a flexible and agnostic DeFi primitive. Due to constraints such as gas and math approximations, there are some limitations built into the protocol.

### V1 Limits

**ERC20 Tokens**

ERC20 compliance: pool tokens have to be ERC20 compliant. Bronze does not support ERC20 tokens that do not return `bools` for `transfer` and `transferFrom`. _\*\*_There are no upgrade mechanisms in the contracts to allow for token upgrades. Any upgrade will need to be manually coordinated and moved into new pools.

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

### V2 Limits

Note - these are current values, and may change until release.

**ERC20 Tokens**

ERC20 compliance: pool tokens should generally be ERC20 compliant, though it is more flexible than V1 in some areas \(e.g., it does not check for `bools` for `transfer` and `transferFrom`**\).** There are no upgrade mechanisms in the contracts to allow for token upgrades. Any upgrade will need to be manually coordinated and moved into new pools.

Tokens that have internal transfer fees or other non-standard balance updates may create unexpected arbitrage opportunities. Ex: DGX has both a demurrage and a transfer fee that will change a pool's balance compared to the internal accounting balance. These must be handled through special pool logic on V1, and are not supported on V2.

**Minimum Bound Tokens - 2**

Weighted and Stabe pools must contain at least two tokens.

**Maximum Bound Tokens - 16**

The maximum number of tokens that can be in a given Weighted pool is 16. Stable pools currently have a practical limitation of 4, but this is still in development. There is no theoretically limit on the number of tokens new pools can support - though the math \(and block gas limit\) will impose practical restrictions. For instance, the Weighted Pool math imposes an upper limit of 100 tokens.

**Maximum Swap In/Out Ratio - 0.3**

A maximum swap in ratio of 0.3 means a user can only swap in less than 30% of the current balance of `tokenIn` for a given pool, or swap out less than 30% of the current balance of `tokenOut`.

**Minimum Swap Fee - 0**

It is possible to have a zero swap fee on V2.

**Maximum Swap Fee - 10%**

This is to prevent malicious pool controllers from setting predatory trading fees. \(For instance, a pool controller could front-run a large trade and set the fee to 99%.\) No one wants to be [this guy](https://etherchain.org/tx/c215b9356db58ce05412439f49a842f8a3abe6c1792ff8f2c3ee425c3501023c). Note that V2 also charges a protocol fee, up to 50% of the pool's swap fee. For instance if the pool swap fee is 1%, and the protocol swap fee is also 1%, the total fee paid by the trader is 0.01 + 0.01 \* 0.01 = 1.01%.

**Minimum Balance - it's complicated**

This is done differently in the V2 math. Balances can't be zero, but the mechanism keeping them non-zero is different - mainly burning a small amount of the initial BPT on creation.

**Min/Max Initial BPT Supply - it's complicated**

The initial supply is computed in the new V2 math, and is not passed in by the user. The protocol also burns an initial amount to prevent rounding/boundary issues.

#### Protocol Fees

The max protocol swap fee is 50% of the pool's swap fee \(which is in turn capped at 10%\). The max protocol withdrawal fee is 0.5%. The max flash loan fee is 1%.

