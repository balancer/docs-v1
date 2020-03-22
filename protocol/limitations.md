# Limitations

Balancer is meant to be a flexible and agnostic primitive. Due to constraints such as gas and math approximations, there are some limitations built into the protocol.

**ERC20 Tokens**

ERC20 compliance: pool tokens have to be ERC20 compliant. Bronze does not support ERC20 tokens that do not return `bools` for `transfer` and `transferFrom`. ****There are no upgrade mechanisms in the contracts to allow for token upgrades. Any upgrade will need to be manually coordinated and moved into new pools.

 **Maximum Bound Tokens - 8**

The max number of tokens that can be in a given pool is 8.

**Maximum Swap In Ratio - 1/2**

A maximum swap in ratio of 0.50 means a user can only swap in less than 50% of the current balance of `tokenIn` for a given pool

**Maximum Swap Out Ratio - 1/3**

A maximum swap out ratio of 1/3 means a user can only swap out less than 33.33% of the current balance of `tokenOut` for a given pool

**Minimum Swap Fee - 0.0001%**

There is a minimum swap fee of 0.0001% \(or a hundredth of a basis point\) to counteract any unfavorable pool rounding.

**Maximum Swap Fee - 10%**

This is to prevent malicious pool controllers from setting predatory trading fees.

**Minimum Balance - \(10^18\) / \(10^12\)**

The minimum balance of any token in a pool is 10^6 wei. **Important**: this is agnostic to token decimals and may cause issues for tokens with less than 6 decimals. Also note that this is only enforced on initial token binding. Future exits can potentially bring the pool below the minimum balance threshold and users should be aware of potential rounding errors.



