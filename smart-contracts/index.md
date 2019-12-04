# Summary

## Note about releases

The **🍂Bronze Release🍂** is the first of 3 planned releases of the Balancer Protocol. Bronze emphasizes code clarity for audit and verification, and does not go to great lengths to optimize for gas.

The **❄️Silver Release❄️** will bring many gas optimizations and architecture changes that will reduce transaction overhead and enable more flexibile usage patterns.

The **☀️Golden Release☀️** will introduce a few final features that will tie the whole system together.

Unless noted otherwise, this document describes the contract architecture for the **🍂Bronze Release🍂**. We have attempted to make note of the functionality that will change for future releases, but it is possible for _any aspect_ of the design to change in both very large and also very subtle ways.

Please take care to when interacting with a Balancer pool to ensure you know which release it is associated with. Objects in the Balancer system provide `getColor() returns (bytes32)`, which returns a left-padded ASCII representation of the all-caps color word, ie, `bytes32("BRONZE")`.

### Events

`LOG_CALL` is an anonymous event which uses the function signature as the event signature. It is fired by all stateful functions.

```text
event LOG_CALL(
    bytes4  indexed sig,
    address indexed caller,
    bytes           data
) anonymous;
```

`LOG_SWAP` is fired \(along with `LOG_CALL`\) for all [swap variants](index.md).

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

