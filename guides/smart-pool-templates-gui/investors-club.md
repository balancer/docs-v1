# Investors' Club

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/smart-pool-templates-gui/investors-club)

## Investors' Club

With Smart Pools, it is possible to restrict liquidity providers to a defined group. For instance, accredited investors, or those who have complied with a KYC process \(e.g., for STOs\). However, the privileged addresses could be determined by anything - holders of certain tokens, those who participated in community building or voting, winners of a "lottery" for investment slots, etc.

Legend**: required;** ~~not required;~~ _optional_

**Rights configuration:**

* _canPauseSwapping_
* ~~canChangeSwapFee~~
* ~~canChangeWeights~~
* ~~canAddRemoveTokens~~
* **canWhitelistLPs**
* _canChangeCap_

The mechanism for this is the LP whitelist, which has to be enabled. The controller can add or remove addresses from this list, and only those addresses are permitted to join pools. \(NB: the pool controller cannot prevent swaps - other than by using the pauseSwapping right, which halts it altogether for everyone. Also, the pool tokens are ERC-20 conforming, so do not prevent transfers. It's the LP's responsibility to keep track of their tokens, and not transfer them to unauthorized parties.\)

Depending on the use case, the pool operator might need to pause swapping \(e.g., during a signup or redemption period\). If it is an STO, there might also be hard- or soft caps on the value of the pool. If this is the case, the cap right would also need to be enabled. See [Experimental](experimental.md).

