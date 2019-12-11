# Limitations

Balancer is meant to be a flexible and agnostic primitive. Due to constraints such as gas and math approximations, there are some limitations built into the protocol.



 **Maximum Bound Tokens - 8**

The max number of tokens that can be in a given pool is 8.

**Maximum Swap In Ratio - 1/2**

A maximum swap in ratio of 0.50 means an user can only swap in less than 50% of the current balance of `tokenIn`

**Maximum Swap Out Ratio - 1/3**

A maximum swap out ratio of 1/3 means an user can only swap out less than 33.33% of the current balance of `tokenOut`

**Maximum Swap Fee - 10%**

This is to prevent malicious pool controllers from setting predatory trading fees.

**Minimum Balance - 0.0001%**

The minimum balance of any token in a pool is 0.0001% \(or a hundredth of a basis point\).

**Maximum Balance - 10^12**

The maximum balance of any token in a pool is 10^12.



