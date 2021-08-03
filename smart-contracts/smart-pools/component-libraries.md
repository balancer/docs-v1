# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/smart-pools/component-libraries)

# Component Libraries

The Configurable Rights Pool uses several externally linked libraries to implement all its functionality, detailed below.

* **BalancerConstants** - constant state variables used throughout
* **BalancerSafeMath** - similar to Open Zeppelin SafeMath, but normalized so that "1" = 10^18, to allow fractional arithmetic \(e.g., on weights\)
* **RightsManager** - defines a struct of boolean values, corresponding to each right; the Configurable Rights Pool stores this struct in storage.
* **SmartPoolManager** - factors out computationally intensive functions, mainly to reduce the bytecode size of the Configurable Rights Pool, to keep it deployable
* **SafeApprove** - an internal library \(adapted from PieDAO\) to enable pools to contain ERC20 tokens that require approve calls to be made from a base of 0 \(e.g., KNC\)

The Configurable Rights Pool contains getter functions that return the addresses of these libraries; a bit of future-proofing, so that clients can implement versioning.

