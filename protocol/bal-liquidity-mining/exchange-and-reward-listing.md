# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/bal-liquidity-mining/exchange-and-reward-listing)

# Exchange and Reward Listing

Token listings are managed in [this repository](https://github.com/balancer-labs/assets), with the following categories:

* `eligible.json`: assets eligible for BAL mining as per weekly proposals
* `listed.json`: assets listed on balancer.exchange
* `ui-not-eligible.json`: assets vetted by community members
* `untrusted.json`: assets that are incompatible with Balancer

There are two kinds of token "listings" on Balancer. The first is listing on the [Exchange](https://balancer.exchange/#/swap). Listed tokens appear on the main page as swapping options. It is possible to access unlisted pairs through a "deep link" with their addresses \(balancer.exchange/\#/swap/&lt;address1&gt;/&lt;address2&gt;\), but it display unlisted tokens as addresses, and with a warning.

There is no formal process for listing tokens on the Balancer exchange UI. That is up to the team's discretion and relies on internal factors around trading volume, usage, and legitimacy. \(As noted above, it's always possible to trade tokens via contract address.\)

The second kind of listing is eligibility for BAL rewards. New tokens are listed \(and occasionally removed\) through a weekly governance process There is a streamlined process for simply listing a new token - tokens that meet a set of technical criteria below can be approved without requiring community vote. Only increasing the cap, introducing "controversial" or non-conforming tokens, or adjusting the reward process or its parameters requires a vote.

To request approving your token for BAL rewards, just post to \#token-requests on the [Discord](https://discord.gg/ARJWaeF) channel. Each weekly reward period begins at 00:00 UTC on Monday; any requests approved after then will take effect the following week.

The listing criteria below are taken from this [accepted proposal](https://forum.balancer.finance/t/proposal-to-update-the-whitelist-process/217/4). \(We will endeavor to keep this up-to-date, but see the [Balancer Forum](https://forum.balancer.finance/) for the very latest.\)

1. The token’s smart contract must be verified on [Etherscan](https://etherscan.io/). Neglecting this small degree of transparency adds unnecessary friction to the process of vetting for the remaining criteria.
2. The token must conform to the ERC-20 interface described in [EIP-20](https://eips.ethereum.org/EIPS/eip-20). Namely, the functions `transfer()`, `transferFrom()`, and `approve()` must return booleans.
3. The token’s `transfer()` and `transferFrom()` implementations must exhibit the expected behavior - namely, transferring N tokens from one address to another. Certain divergences from this behavior, such as transfer fees, can cause issues with Balancer pools, and these tokens will be rejected.
4. The token’s `approve()` implementation must exhibit the expected behavior - namely, it must allow for infinite approval, or the token cannot be sold using the Balancer exchange proxy.
5. The token must not be vulnerable to the so-called "`gulp()` attack," which is exposed when a pool’s token balance changes unbeknownst to the pool. For now, known cases include tokens that charge a transfer fee \(as described in \#3\) and tokens that periodically rebase. A rebasing token utilizing an atomic rebase+gulp should be safe from such attacks, but none has yet been discovered. **\(Added note:** Ampleforth released a pool with this behavior, though it was implemented through a Balancer Pool subclass, not the token contract.\)
6. The token must not possess any mechanisms which, when combined with a Balancer pool, result in material losses to the principal value of the pooled token. This criterion covers all value-loss edge cases which may be difficult to anticipate but can still preclude a token’s whitelisting. Examples will be added as they are discovered; the only currently known example is [VBZRX](https://etherscan.io/address/0xB72B31907C1C95F3650b64b2469e08EdACeE5e8F), whose `claim()` mechanism entitles token holders to receive BZRX airdrops on each token transfer. In isolation, this mechanism is perfectly safe, but if a token is airdropped to a Balancer pool whose asset list does not contain said token, then the airdropped tokens are forever locked in the pool contract and cannot be recovered by anyone \(including Balancer Labs\).
7. The token must have a price feed accessible via CoinGecko’s API, e.g. [this example link](https://api.coingecko.com/api/v3/simple/token_price/ethereum?contract_addresses=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2&vs_currencies=usd) for WETH. This is instrumental to calculating a pool’s eligibility for BAL rewards. If a token meets all of the remaining criteria but not this criterion, it can be added to the UI whitelist and reconsidered for the mining whitelist once the price feed becomes available.

