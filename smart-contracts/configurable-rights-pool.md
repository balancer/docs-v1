# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/configurable-rights-pool)

# Configurable Rights Pool

The Balancer **Configurable Rights Pool** is the reference implementation of a smart contract-controlled Balancer Pool. It is flexible enough to be used directly to create customizable Smart Pools. Like the Core Pool \(BPool\), it is created from a factory - and as the name implies, its functionality can be customized to meet the needs of your project.

It is also designed to be easily extensible, and we showcase examples of Configurable Rights Pool extensions in [Smart Pool Templates](smart-pool-templates.md). These are projects with specific needs and custom logic, which need to override and alter core functionality.

For example, the first template is [Ampleforth's](https://www.ampleforth.org/) Elastic Supply Pool, which features a different mechanism for updating weights, consistent with its daily rebase operation. We will add more templates as we discover further interesting use cases! \(If you have a suggestion, we'd love to hear from you on the smart-pool-dev channel on [Discord](https://discord.gg/qjFcczk).\)

You will find the addresses for deployed contracts on all networks [here](addresses.md).

### How is a Smart Pool different from a Core Pool?

As explained in [Core Concepts](../protocol/concepts.md), a core Balancer Pool's parameters can be changed at will while in the "controlled" state - when only the pool creator can add liquidity. The only way to "open" the pool to outside investment is to "finalize" it - after which all pool parameters are fixed. This is great for security and trust minimization - but what if you want to do liquidity bootstrapping, dynamically adjust swap fees, handle a token with unique properties \(such as AMPL\), or implement a complex investment strategy? For that, you'll need to change parameters dynamically on "live" pools - and that requires a Smart Pool.

In summary, a Core Pool has "all rights" while controlled, and then "no rights" after finalization. A Configurable Rights Pool has whatever rights you give it, for all time. So what are these rights?





