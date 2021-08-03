# BAL for Gas

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/core-concepts/bal-balancer-governance-token/bal-for-gas)

## BAL for Gas

Following a 6 week pilot, on March 7th 2021 the Balancer community approved a campaign to increase BAL distribution. To ensure lasting robustness of the governance process, both liquidity providers and traders should have a say in how the protocol evolves.

While liquidity providers receive BAL as a function of the amount of liquidity provided in the system, traders earn BAL for swapping tokens on the Balancer Exchange dApp.

Every eligible trade made through the Balancer Exchange Proxy results in some BAL being allocated to the address \(EOAs only\) that sent the transaction. An eligible trade is a trade containing one or more eligible swaps, where an eligible swap is one between any two tokens on the [whitelist](https://github.com/balancer-labs/assets/blob/master/lists/eligible.json). Claims are made available at the [BAL claims interface](https://claim.balancer.finance/) on Wednesday \(UTC time\) following the end \(00:00 UTC Monday\) of the weekly period in which the trade occurred.

The amount of BAL awarded to a trade is a function of:

* the number of eligible swaps in the trade \(N\), which determines a number of gas units \(G\):
  * trades on Balancer V1:
    * 1 swap: `G=130000` gas units
    * 2 swaps: `G=220000` gas units
    * 3 swaps: `G=300000` gas units
    * 4+ swaps: `G=400000` gas units
  * trades on Balancer V2:
    * 1 swap: `G=90000` gas units
    * 2+ swaps: `G=140000` gas units
* the median gas price \[1\] of the block the transaction was included in \(M\); and 
* the BAL/ETH price provided by the CoinGecko closest to the block time \(P\).

The amount of BAL to be received by the user for a trade is computed as `G*M/P`

Because this program only partially covers gas costs \(with very specific caps to prevent attempts at gaming the system\), and doesn’t cover trading fees, it does not incentivize wash trading or trading any more than a user was initially planning. Estimates are provided on the UI, but actual values are computed off-chain weekly using an [open source script](https://github.com/balancer-labs/bal-mining-scripts/), so that any suspicious activity can be filtered out.

80,000 BAL from the Ecosystem Fund have been allocated to the “BAL for Gas’’ campaign, starting on March 8th 2021. There is no fixed time period; the budget is consumed on a first-come, first-served basis. When the budget is exhausted, the campaign is suspended until it is replenished by BAL governance.

### Notes

\[1\] Transactions sent by the miner of the block or having a gas price of 10 Gwei or less are not considered for the purposes of determining the median gas price of the block

### References

* [BAL for Gas proposal](https://forum.balancer.finance/t/proposal-bal-for-gas/1437)
* [Original pilot](https://forum.balancer.finance/t/proposal-balancer-exchange-gas-reimbursement/705) and [Expansion of the list of eligible tokens](https://forum.balancer.finance/t/proposal-expand-the-exchange-gas-reimbursement-to-all-whitelisted-tokens/799)
* [Extension of the pilot, 30k BAL budget](https://forum.balancer.finance/t/proposal-extend-the-exchange-gas-reimbursement-program-4-weeks/1121)
* [Additional 50k BAL budget](https://forum.balancer.fi/t/proposal-bal-for-gas-replenish-budget-w-50-000-bal/1695)
* [BAL for Gas on V2 params](https://forum.balancer.fi/t/proposal-bal-for-gas-on-balancer-v2/1861)

