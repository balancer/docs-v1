# BPTs

All pools in Balancer are also ERC20 tokens known as BPTs \(Balancer pool tokens\) which represent proportional ownership in the pool's liquidity. When users add liquidity through `joinPool` or `joinswap*` a function they receive BPTs proportional to the amount of assets they are adding.

## Notes

BPTs are an opinionated ERC20 token implementation and have a few subtle differences. `transferFrom` from itself does not require a previous allowance.

