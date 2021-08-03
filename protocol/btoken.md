# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/btoken)

# BPTs

All pools in Balancer are also ERC20 tokens known as BPTs \(Balancer Pool Tokens\), which represent proportional ownership in the pool's liquidity. When users add liquidity through `joinPool` or `joinswap*` they receive BPTs proportional to the amount of assets they are adding to the pool.

## Notes

BPTs are an opinionated ERC20 token implementation, and have a few subtle differences. `transferFrom` from itself does not require a previous allowance.

