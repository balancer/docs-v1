# Math

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/core-concepts/protocol/index/)

## Math

The [Balancer whitepaper](https://balancer.finance/whitepaper.html) describes a set of formulas derived from the value function for interacting with the protocol. The formulas in the Theory section are sufficient to describe the functional specification, but they are not straightforward to implement for the EVM, in part due to a lack of mature fixed-point math libraries.

Our implementation uses a combination of a few algebraic transformations, approximation functions, and numerical hacks to compute these formulas with bounded maximum error and reasonable gas cost.

### Exponentiation Approximation

#### Spot Price

$$
SP^o_i = \frac{ \frac{B_i}{W_i} }{ \frac{B_o}{W_o} }
$$

Where:

* _Bi_ is the balance of token _i_, the token being sold by the trader which is going _into_ the pool.
* _Bo_ is the balance of token _o_, the token being bought by the trader which is coming _out_ of the pool.
* _Wi_ is the weight of token _i_
* _Wo_ is the weight of token _o_

When we consider swap fees, we do exactly the same calculations as without fees, but using $$A_i \cdot (1-swapFee)$$ instead of $$A_i$$. This strategy is referred to as charging fees "on the way in." With the swap fee, the spot price increases. It then becomes:

$$
SP^o_i = \frac{ \frac{B_i}{W_i} }{ \frac{B_o}{W_o} } \cdot \frac{1}{(1-swapFee)}
$$

#### Out-Given-In

In the [Whitepaper](https://balancer.finance/whitepaper/), we derive the following formula to calculate the amount of tokens out –$$A_o$$– a trader gets in return for a given amount of tokens in –$$A_i$$, considering a Balancer pool without any swap fees:

$$
A_{o} = B_{o}  \cdot \left(1 - \left(\frac{B_{i}}{B_{i}+A_{i}}\right)^{\frac{W_{i}}{W_{o}}}\right)
$$

To take into account the swap fees charged by the Balancer pool, we replace$$A_i$$with$$A_i \cdot (1-swapFee)$$. This is known as charging the fees "on the way in"

$$
A_{o} = B_{o}  \cdot \left(1 - \left(\frac{B_{i}}{B_{i}+A_{i} \cdot (1-swapFee)}\right)^{\frac{W_{i}}{W_{o}}}\right)
$$

#### In-Given-Out

In the [Whitepaper](https://balancer.finance/whitepaper/), we derive the following formula for the amount of tokens in –$$A_i$$– a trader needs to swap to get a desired amount$$A_o$$of tokens out in return, considering a Balancer pool without any swap fees:

$$
A_{i} = B_{i} \cdot \left(\left(\frac{B_{o}}{B_{o}-A_{o}}\right)^{\frac{W_{o}}{W_{i}}}-1\right)
$$

Since $$A_i$$ is the amount the user has to swap to get a desired amount out $$A_o$$, all we have to do to include swap fees is divide the formula above by $$(1-swapFee)$$. This is because we know the fee charged on the way in will multiply that amount$$A_i$$ by $$(1-swapFee)$$. This will cross out both terms $$(1-swapFee)$$ and the amount out will be $$A_o$$ as desired:

$$
A_{i} = B_{i} \cdot \left(\left(\frac{B_{o}}{B_{o}-A_{o}}\right)^{\frac{W_{o}}{W_{i}}}-1\right) \cdot \frac{1}{(1-swapFee)}
$$

#### All-Asset Deposit/Withdrawal

Anyone can be issued Balancer pool tokens \(provided the pool is finalized\) by depositing proportional amounts of each of the assets contained in the pool. So, for each token _k_ in the pool, the amounts of token _k_ –$$D_k$$– that need to be deposited for someone to get $$P_{issued}$$pool tokens are:

$$
D_k = \left(\frac{P_{supply}+P_{issued}}{P_{supply}}-1\right) \cdot B_k
$$

Conversely, if a user wants to redeem their pool tokens to get their proportional share of each of the underlying tokens in the pool, the amounts of token k –$$A_k$$– a user gets for redeeming $$P_{redeemed}$$pool tokens will be:

$$
A_k = \left(1-\frac{P_{supply}-P_{redeemed}}{P_{supply}}\right) \cdot B_k
$$

All Balancer Protocol smart contracts were coded supporting a protocol-level exit fee to be charged that goes to Balancer Labs for supporting the development of the protocol. However, after careful consideration the Balancer Labs team decided to launch the first version of Balancer without any protocol fees whatsoever. \(For technical reasons, this is unlikely to change.\)

#### Single-Asset Deposit / Withdrawal

**Single-Asset Deposit**

In the [Whitepaper](https://balancer.finance/whitepaper/), we derive the following formula for the amount of pool tokens –$$P_{issued}$$– a liquidity provider gets in return for depositing an amount $$A_t$$of a single token _t_ present in the pool:

$$
P_{issued} = P_{supply} \cdot \left(\left(1+\frac{A_t}{B_t}\right)^{W_t} -1\right)
$$

Since Balancer allows for depositing and withdrawing liquidity to Balancer pools using only one of the tokens present in the pool, this could be used to do the equivalent of a swap: provide liquidity depositing token A, and immediately withdraw that liquidity in token B. Therefore a swap fee has to be charged, proportional to the tokens that would need to be swapped for an all-asset deposit.

Another justification for charging a swap fee when a liquidity provider does a single-asset deposit is that they are getting a share of a pool that contains a basket of different assets. So what they are really doing is trading one of the pool assets \(the token _t_ being deposited\) for proportional shares of all the pool assets.

Since the pool already has a share of its value in token _t_, represented by the weight$$W_t$$, it only makes sense to charge a swap fee for the remaining portion of the deposit $$A_t \cdot(1 - W_t)$$

The formula then becomes:

$$
P_{issued} = P_{supply} \cdot \left(\left(1+\frac{\left(A_t-A_t\cdot(1 - W_t)\cdot swapFee\right)}{B_t}\right)^{W_t} -1\right)
$$

The formula above calculates the amount of pool tokens one receives in return for a deposit of a given amount of a single asset. We also allow for users to define a given amount of pool tokens they desire to get – $$P_{issued}$$– and calculate what amount of tokens _t_ is needed – $$A_t$$:

$$
A_t = B_t \cdot \left(\left(1+\frac{P_{issued}}{P_{supply}}\right)^{\frac{1}{W_t}} -1\right)
$$

Taking into account the swap fees, we have:

$$
A_t = B_t \cdot \frac{\left(\left(1+\frac{P_{issued}}{P_{supply}}\right)^{\frac{1}{W_t}} -1\right)}{(1 - W_t)\cdot swapFee}
$$

**Single-Asset Withdrawal**

Without considering swap fees, each withdrawal formula is simply the inverse of the corresponding deposit formula. In other words, if you deposit a given amount of token _t_ for pool tokens and then immediately redeem these pool tokens for token _t_, you should receive exactly what you started off with.

The formula without considering swap fees is then:

$$
A_t = B_t \cdot \left(1-\left(1-\frac{P_{redeemed}}{P_{supply}}\right)^\frac{1}{W_t}\right)
$$

Where $$A_t$$ is the amount of token _t_ one receives when redeeming $$P_{redeemed}$$pool tokens.

Considering swap fees, we have the following:

$$
A_t = B_t \cdot \left(1-\left(1-\frac{P_{redeemed}}{P_{supply}}\right)^\frac{1}{W_t}\right)\cdot \left(1-(1 - W_t)\cdot swapFee\right)
$$

If there were an exit fee, it would be taken from the amount of tokens redeemed $$P_{redeemed}$$but as mentioned above this fee is zero in the first version of Balancer.

Balancer also allows for a liquidity provider to choose a desired amount of token t, $$A_t$$, they would like to withdraw from the pool, and calculates the necessary amount of pool tokens required for that,$$P_{redeemed}$$. The formula without considering swap fees is:

$$
P_{redeemed} = P_{supply} \cdot \left(1-\left(1-\frac{A_t}{B_t}\right)^{W_t} \right)
$$

Where $$A_t$$ is the amount of token _t_ one receives when redeeming $$P_{redeemed}$$pool tokens.

Considering swap fees, we have the following:

$$
P_{redeemed} = P_{supply} \cdot \left(1-\left(1-\frac{\frac{A_t}{\left(1-(1 - W_t)\cdot swapFee\right)}}{B_t}\right)^{W_t} \right)
$$

