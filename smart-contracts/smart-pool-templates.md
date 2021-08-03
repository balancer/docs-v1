# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/smart-pool-templates)

# Smart Pool Templates

Balancer provides templates at two levels. At the contract level, templates are examples of how to extend the core Smart Pool contract for custom use cases \(see below\).

For most use cases, you can just use the Configurable Rights Pool directly, and need only to customize the settings. The [Smart Pool Templates](../guides/smart-pool-templates-gui/) in the guides section show examples of how to configure smart pools for different expected use cases.

Smart Pool Template contracts are example subclasses of the Configurable Rights Pool, illustrating how particular use cases might be implemented. The first is the Ampleforth-inspired Elastic Supply Pool.

Updating weights in a regular Configurable Rights Pool transfers tokens \(to keep prices stable\). In the case of AMPL, a combination of "rebasing" \(altering the token supply\) and changing weights accomplishes this without transferring tokens. In Elastic Supply Pools, weights are "resynced" instead of "updated."

To disallow both kinds of updates in the same pool, the Elastic Supply Pool overrides and reverts the existing update functions, and implements the new resync\(\) method.



