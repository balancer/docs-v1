# Glossary

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/core-concepts/protocol/glossary)

## Glossary

* **Core Pool**: A `BPool` contract object - this is the "base" pool that actually holds the tokens
* **Balance**: The total token balance of a pool. Does not refer to any user balance.
* **Denorm**: Denormalized weight. Weights on a BPool, though often displayed as percentages, are configured and stored in their **denormalized** form. For instance, in a two-token pool with denormalized weights of A=38 and B=2, token A's percentage weight would be 38/\(38+2\), or 95%. Conversely, token B's proportion would be 2/\(38+2\), or 5%.
* **Controller**: The pool's "owner"; an address that can call `CONTROL` capabilities.
* **Factory**: The official BPool factory. Pools deployed from this factory appear on Balancer user interfaces \(e.g., the [Exchange](https://balancer.exchange/#/swap) and [Pool Manager](https://pools.balancer.exchange/#/)\).
* **Smart Pool**: A contract that owns \(i.e., is the **controller\)**, **\*\*of a** Core ****Pool\*\*. Much more about these later.

