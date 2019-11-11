# Core Concepts

## Terminology

* Pool: A `BPool` contract object
* Balance: The total token balance of a pool. Does not refer to any user balance.
* Denorm: Denormalized weight. Weights on a BPool are configured and stored in their **denormalized** form.
* Controller: The pool's "owner"; an address that can call `CONTROL` capabilities.
* Factory: The official BPool factory. Pools deployed from this factory appear in tools and explorers for Balancer.

## Pool Lifecycle

Any user can create a new pool by calling `newBPool()` on the `BFactory` contract. The caller is set as the `controller` or pool owner.

Pools can exist in one of two states: `controlled` or `finalized`. Pools start in a controlled state and the controller may choose to make the pool finalized by calling `finalize()`. While in a controlled state, outside actors cannot add liqudity. A controlled state allows the controller to set the pool's tokens and weights.

The table below illustrates pool states and the allowed calls:

<table>
  <thead>
    <tr>
      <th style="text-align:left">State</th>
      <th style="text-align:left">Allowed state update calls</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>controlled</code>
      </td>
      <td style="text-align:left">
        <p><code>setSwapFee()</code>
        </p>
        <p><code>setController()</code>
        </p>
        <p><code>setPublicSwap()</code>
        </p>
        <p><code>bind()</code>
        </p>
        <p><code>rebind()</code>
        </p>
        <p><code>unbind()</code>
        </p>
        <p><code>finalize()</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>finalized</code>
      </td>
      <td style="text-align:left">
        <p><code>setController()</code>
        </p>
        <p><code>joinPool()</code>
        </p>
        <p><code>exitPool()</code>
        </p>
        <p><code>swap_ExactAmountIn()</code>
        </p>
        <p><code>swap_ExactAmountOut</code>
        </p>
        <p><code>swap_ExactMarginalPrice()</code>
        </p>
        <p><code>joinswap_ExternAmountIn()</code>
        </p>
        <p><code>joinswap_PoolAmountOut()</code>
        </p>
        <p><code>exitswap_PoolAmountIn()</code>
        </p>
        <p><code>exitswap_ExternAmountOut()</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>### Smart Contract Owned Controlled Pools

One very powerful feature of Balancer is the concept of smart-contract owned controlled pools.

#### Conceptual Capabilities:

* `SWAP`  \(`swap_*`, `joinswap_*`, `exitswap_*`\)
* `JOIN` \(`joinPool`, `joinswap_*`\)
* `EXIT` \(`exitPool`, `exitswap_*`\) &lt;-- always public
* `CONTROL` \(`bind`, `unbind`, `rebind`, `setFees`, `finalize`\)
* `FACTORY` \(`collect`\)

Notice that e.g. `joinswap` requires both `JOIN` and `SWAP`.

