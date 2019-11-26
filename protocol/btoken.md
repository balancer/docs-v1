# BTokens

All pools in Balancer are also ERC20 tokens known as BTokens which represent proportional ownership in the pool's liquidity.

## Notes

BTokens are an opinionated token implementation and have a few subtle differences. `transferFrom` from itself does not require a previous allowance.

