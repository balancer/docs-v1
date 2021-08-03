# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/smart-contracts/core-contracts/)

# Core Contracts

## Note about releases

The **🍂Bronze Release🍂** is the first of 3 planned releases of the Balancer Protocol. Bronze emphasizes code clarity for audit and verification, and does not go to great lengths to optimize for gas.

The **❄️Silver Release❄️** will bring many gas optimizations and architecture changes that will reduce transaction overhead and enable more flexible usage patterns.

The **☀️Gold Release☀️** will introduce a few final features that will tie the whole system together.

Unless noted otherwise, this document describes the contract architecture for the **🍂Bronze Release🍂**. We have attempted to make note of the functionality that will change for future releases, but it is possible for _any aspect_ of the design to change in both very large and also very subtle ways.

Please take care when interacting with a Balancer pool to ensure you know which release it is associated with. Objects in the Balancer system provide `getColor() returns (bytes32)`, which returns a left-padded ASCII representation of the all-caps color word, ie, `bytes32("BRONZE")`.

Similarly, Configurable Rights Smart Pools have getter functions to return the version of the Smart Pool Manager, Rights Manager, and Safe Math library versions they were linked to on deployment. Of course, you can also retrieve the underlying Core Pool controlled by the Smart Pool, and call getColor\(\) on it.

This is one reason the CRPFactory takes a BFactory parameter - in the future, passing a Silver or Gold BFactory would create an underlying pool corresponding to that version.

### Events

`LOG_CALL` is an anonymous event which uses the function signature as the event signature. It is fired by all stateful functions. The following applies to Balancer Core pools. Smart Pools have similar events and modifiers.

```text
event LOG_CALL(
    bytes4  indexed sig,
    address indexed caller,
    bytes           data
) anonymous;
```

`LOG_SWAP` is fired \(along with `LOG_CALL`\) for all [swap variants]().

```text
event LOG_SWAP(
    address indexed caller,
    address indexed tokenIn,
    address indexed tokenOut,
    uint256         tokenAmountIn,
    uint256         tokenAmountOut
);
```

`LOG_JOIN` and `LOG_EXIT` are fired for each individual token join / exit

```text
event LOG_JOIN(
    address indexed caller,
    address indexed tokenIn,
    uint256         tokenAmountIn
);

event LOG_EXIT(
    address indexed caller,
    address indexed tokenOut,
    uint256         tokenAmountOut
);
```

### Mutex

All stateful functions use either a `lock` or `viewlock` function modifier. A mutex places a lock on contract state and prevents any sort of re-entrancy.

```text
modifier _lock_() {
    require(!_mutex, "ERR_REENTRY");
    _mutex = true;
    _;
    _mutex = false;
}

modifier _viewlock_() {
    require(!_mutex, "ERR_REENTRY");
    _;
}
```

