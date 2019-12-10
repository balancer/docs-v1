# Math

The [Balancer whitepaper](https://balancer.finance/whitepaper.html) describes a set of formulas derived from the value function for interacting with the protocol. The formulas in the Theory section are sufficient to describe the functional specification, but they are not straightforward to implement for the EVM, in part due to a lack of mature fixed-point math libraries.

Our implementation uses a combination of a few algebraic transformations, approximation functions, and numerical hacks to compute these formulas with bounded maximum error and reasonable gas cost.

## Exponentiation Approximation

[Approxing](approxing.md)

### Spot Price

$$
SP^o_i = \frac{ \frac{B_i}{W_i} }{ \frac{B_o}{W_o} }
$$

Where:

* _Bi_ is the balance of token _i_, the token being sold by the trader which is going _into_ the pool.
* _Bo_ is the balance of token _o_, the token being bought by the trader which is going _out_ of the pool.
* _Wi_ is the weight of token _i_
* _Wo_ is the weight of token _o_

When we consider swap fees, we do exactly the same calculations as without fees but using $$A_i \cdot (1-swapFee)$$ instead of $$A_i$$. This strategy is referred to as charging fees "on the way in". With the swap fee, the spot price increases. It becomes then:

$$
SP^o_i = \frac{ \frac{B_i}{W_i} }{ \frac{B_o}{W_o} } \cdot \frac{1}{(1-swapFee)}
$$

```text
/**********************************************************************************************
// _calc_SpotPrice                                                                           //
// sP = spotPrice                                                                            //
// bI = tokenBalanceIn                ( bI / wI )         1                                  //
// bO = tokenBalanceOut         sP =  -----------  *  ----------                             //
// wI = tokenWeightIn                 ( bO / wO )     ( 1 - sF )                             //
// wO = tokenWeightOut                                                                       //
// sF = swapFee                                                                              //
**********************************************************************************************/

function _calc_SpotPrice(
    uint tokenBalanceIn,
    uint tokenWeightIn,
    uint tokenBalanceOut,
    uint tokenWeightOut,
    uint swapFee
)
    internal pure
    returns ( uint spotPrice )
{
    uint numer = bdiv(tokenBalanceIn, tokenWeightIn);
    uint denom = bdiv(tokenBalanceOut, tokenWeightOut);
    uint ratio = bdiv(numer, denom);
    uint scale = bdiv(BONE, bsub(BONE, swapFee));
    return  (spotPrice = bmul(ratio, scale));
}
```

### Out-Given-In

In the whitepaper we derive the following formula that calculates the amount of tokens out –$$A_o$$– a trader gets in return for a given amount of tokens in –$$A_i$$, considering a Balancer pool without any swap fees: 

$$
A_{o} = B_{o}  \cdot \left(1 - \left(\frac{B_{i}}{B_{i}+A_{i}}\right)^{\frac{W_{i}}{W_{o}}}\right)
$$

To take into account the swap fees charged by the Balancer pool, we replace$$A_i$$with$$A_i \cdot (1-swapFee)$$. This is know as charging the fees "on the way in":

$$
A_{o} = B_{o}  \cdot \left(1 - \left(\frac{B_{i}}{B_{i}+A_{i} \cdot (1-swapFee)}\right)^{\frac{W_{i}}{W_{o}}}\right)
$$

```text
/**********************************************************************************************
// _calc_OutGivenIn                                                                          //
// aO = tokenAmountOut                                                                       //
// bO = tokenBalanceOut                                                                      //
// bI = tokenBalanceIn              /      /            bI             \    (wI / wO) \      //
// aI = tokenAmountIn    aO = bO * |  1 - | --------------------------  | ^            |     //
// wI = tokenWeightIn               \      \ ( bI + ( aI * ( 1 - sF )) /              /      //
// wO = tokenWeightOut                                                                       //
// sF = swapFee                                                                              //
**********************************************************************************************/

function _calc_OutGivenIn(
    uint tokenBalanceIn,
    uint tokenWeightIn,
    uint tokenBalanceOut,
    uint tokenWeightOut,
    uint tokenAmountIn,
    uint swapFee
)
    internal pure
    returns ( uint tokenAmountOut )
{
    uint weightRatio    = bdiv(tokenWeightIn, tokenWeightOut);
    uint adjustedIn     = bsub(BONE, swapFee);
         adjustedIn     = bmul(tokenAmountIn, adjustedIn);
    uint y              = bdiv(tokenBalanceIn, badd(tokenBalanceIn, adjustedIn));
    uint foo            = bpow(y, weightRatio);
    uint bar            = bsub(BONE, foo);
         tokenAmountOut = bmul(tokenBalanceOut, bar);
    return tokenAmountOut;
}
```

### In-Given-Out

In the whitepaper we derive the following formula that calculates the amount of tokens in –$$A_i$$– a trader needs to swap to get a desired amount$$A_o$$of tokens out  in return, considering a Balancer pool without any swap fees: 

$$
A_{i} = B_{i} \cdot \left(\left(\frac{B_{o}}{B_{o}-A_{o}}\right)^{\frac{W_{o}}{W_{i}}}-1\right)
$$

Since $$A_i$$ is the amount the user has to swap to get a desired amount out $$A_o$$, all we have to do to include swap fees is divide the formula above by $$(1-swapFee)$$. This is because we know the fee charged on the way in will multiply that amount$$A_i$$ by $$(1-swapFee)$$. This will cross out both terms $$(1-swapFee)$$ and the amount out will be $$A_o$$ as desired:

$$
A_{i} = B_{i} \cdot \left(\left(\frac{B_{o}}{B_{o}-A_{o}}\right)^{\frac{W_{o}}{W_{i}}}-1\right) \cdot \frac{1}{(1-swapFee)}
$$

```text
/**********************************************************************************************
// _calc_InGivenOut                                                                          //
// aI = tokenAmountIn                                                                        //
// bO = tokenBalanceOut               /  /     bO      \    (wO / wI)      \                 //
// bI = tokenBalanceIn          bI * |  | ------------  | ^            - 1  |                //
// aO = tokenAmountOut    aI =        \  \ ( bO - aO ) /                   /                 //
// wI = tokenWeightIn           --------------------------------------------                 //
// wO = tokenWeightOut                          ( 1 - sF )                                   //
// sF = swapFee                                                                              //
**********************************************************************************************/
function _calc_InGivenOut(
    uint tokenBalanceIn,
    uint tokenWeightIn,
    uint tokenBalanceOut,
    uint tokenWeightOut,
    uint tokenAmountOut,
    uint swapFee
)
    internal pure
    returns ( uint tokenAmountIn )
{
    uint weightRatio   = bdiv(tokenWeightOut, tokenWeightIn);
    uint diff          = bsub(tokenBalanceOut, tokenAmountOut);
    uint y             = bdiv(tokenBalanceOut, diff);
    uint foo           = bpow(y, weightRatio);
         foo           = bsub(foo, BONE);
         tokenAmountIn = bsub(BONE, swapFee);
         tokenAmountIn = bdiv(bmul(tokenBalanceIn, foo), tokenAmountIn);
    return tokenAmountIn;
}
```

### All-Asset Deposit/Withdrawal

Anyone can be issued Balancer pool tokens \(provided the pool is finalized\) by depositing proportional amounts of each of the assets contained in the pool. So, for each token _k_ in the pool, the amounts of token _k_ –$$D_k$$– that need to be deposited for someone to get $$P_{issued}$$pool tokens are:

$$
D_k = \left(\frac{P_{supply}+P_{issued}}{P_{supply}}-1\right) \cdot B_k
$$

Conversely, if a user wants to redeem their pool tokens to get their proportional share of each of the underlying tokens in the pool, the amounts of token k –$$A_k$$– a user gets for redeeming $$P_{redeemed}$$pool tokens will be:

$$
A_k = \left(1-\frac{P_{supply}-P_{redeemed}}{P_{supply}}\right) \cdot B_k
$$

All Balancer Protocol smart contracts were coded supporting a protocol-level exit fee to be charged that goes to Balancer Labs for supporting the development of the protocol. However, after careful consideration the Balancer Labs team decided to launch the first version of Balancer without any protocol fees whatsoever.

### Single-Asset Deposit / Withdrawal

#### Single-Asset Deposit

In the whitepaper we derive the following formula that calculates the amount of pool tokens –$$P_{issued}$$– a liquidity provider gets in return for depositing an amount $$A_t$$of a single token _t_  present in the pool: 

$$
P_{issued} = P_{supply} \cdot \left(\left(1+\frac{A_t}{B_t}\right)^{W_t} -1\right)
$$

Since Balancer allows for depositing and withdrawing liquidity to Balancer pools using only one of the tokens present in the pool, this could be used to do the equivalent of a swap: provide liquidity depositing token A and immediately withdraw that liquidity in token B. Therefore a swap fee has to be charged for the amount of tokens that are deposited that would have to be swapped for an all-asset deposit. 

Another way of justifying the need for charging a swap fee when a liquidity provider does a single-asset deposit is that they are getting a share of a pool that contains a basket of different assets. So in essence what they are doing is a trade of one of the pool assets \(the token _t_ being deposited\) for all of the pool assets proportionally to their weights. 

Since the pool already has a share of its value in token t, represented by the weight$$W_t$$, it only makes sense to charge a swap fee for the remaining proportion of the deposit $$A_t \cdot(1 - W_t)$$

The formula then becomes:

$$
P_{issued} = P_{supply} \cdot \left(\left(1+\frac{\left(A_t-A_t\cdot(1 - W_t)\cdot swapFee\right)}{B_t}\right)^{W_t} -1\right)
$$

```text
/**********************************************************************************************
// _calc_PoolOutGivenSingleIn                                                                //
// pAo = poolAmountOut         /                                              \              //
// tAi = tokenAmountIn        ///      /     //    wI \      \\       \     wI \             //
// wI = tokenWeightIn        //| tAi *| 1 - || 1 - --  | * sF || + tBi \    --  \            //
// tW = totalWeight     pAo=||  \      \     \\    tW /      //         | ^ tW   | * pS - pS //
// tBi = tokenBalanceIn      \\  ------------------------------------- /        /            //
// pS = poolSupply            \\                    tBi               /        /             //
// sF = swapFee                \                                              /              //
**********************************************************************************************/
function _calc_PoolOutGivenSingleIn(
    uint tokenBalanceIn,
    uint tokenWeightIn,
    uint poolSupply,
    uint totalWeight,
    uint tokenAmountIn,
    uint swapFee
)
    internal pure
    returns (uint poolAmountOut)
{
    // Charge the trading fee for the proportion of tokenAi
    ///  which is implicitly traded to the other pool tokens.
    // That proportion is (1- weightTokenIn)
    // tokenAiAfterFee = tAi * (1 - (1-weightTi) * poolFee);
    uint normalizedWeight = bdiv(tokenWeightIn, totalWeight);
    uint zaz = bmul(bsub(BONE, normalizedWeight), swapFee); 
    uint tokenAmountInAfterFee = bmul(tokenAmountIn, bsub(BONE, zaz));

    uint newTokenBalanceIn = badd(tokenBalanceIn, tokenAmountInAfterFee);
    uint tokenInRatio = bdiv(newTokenBalanceIn, tokenBalanceIn);

    // uint newPoolSupply = (ratioTi ^ weightTi) * poolSupply;
    uint poolRatio = bpow(tokenInRatio, normalizedWeight);
    uint newPoolSupply = bmul(poolRatio, poolSupply);
    poolAmountOut = bsub(newPoolSupply, poolSupply);
    return poolAmountOut;
}
```

The formula above calculates the amount of pool tokens one receives in return for a deposit of a given amount of a single asset. We also allow for users to define a given amount of pool tokens they desire to get – $$P_{issued}$$– and calculate what amount of tokens _t_ is needed – $$A_t$$:

$$
A_t = B_t \cdot \left(\left(1+\frac{P_{issued}}{P_{supply}}\right)^{\frac{1}{W_t}} -1\right)
$$

Taking into account the swap fees we have:

$$
A_t = B_t \cdot \frac{\left(\left(1+\frac{P_{issued}}{P_{supply}}\right)^{\frac{1}{W_t}} -1\right)}{(1 - W_t)\cdot swapFee}
$$

```text
/**********************************************************************************************
// _calc_SingleInGivenPoolOut                                                                //
// tAi = tokenAmountIn              //(pS + pAo)\     /    1    \\                           //
// pS = poolSupply                 || ---------  | ^ | --------- || * bI - bI                //
// pAo = poolAmountOut              \\    pS    /     \(wI / tW)//                           //
// bI = balanceIn          tAi =  --------------------------------------------               //
// wI = weightIn                              /      wI  \                                   //
// tW = totalWeight                          |  1 - ----  |  * sF                            //
// sF = swapFee                               \      tW  /                                   //
**********************************************************************************************/
function _calc_SingleInGivenPoolOut(
    uint tokenBalanceIn,
    uint tokenWeightIn,
    uint poolSupply,
    uint totalWeight,
    uint poolAmountOut,
    uint swapFee
)
    internal pure
    returns (uint tokenAmountIn)
{
    uint normalizedWeight = bdiv(tokenWeightIn, totalWeight);
    uint newPoolSupply = badd(poolSupply, poolAmountOut);
    uint poolRatio = bdiv(newPoolSupply, poolSupply);

    //uint newBalTi = poolRatio^(1/weightTi) * balTi;
    uint boo = bdiv(BONE, normalizedWeight); 
    uint tokenInRatio = bpow(poolRatio, boo);
    uint newTokenBalanceIn = bmul(tokenInRatio, tokenBalanceIn);
    uint tokenAmountInAfterFee = bsub(newTokenBalanceIn, tokenBalanceIn);
    // Do reverse order of fees charged in joinswap_ExternAmountIn, this way 
    //  pAo == joinswap_ExternAmountIn(Ti, joinswap_PoolAmountOut(pAo, Ti))
    //uint tAi = tAiAfterFee / (1 - (1-weightTi) * swapFee) ;
    uint zar = bmul(bsub(BONE, normalizedWeight), swapFee);
    tokenAmountIn = bdiv(tokenAmountInAfterFee, bsub(BONE, zar));
    return tokenAmountIn;
}
```

#### Single-Asset Withdrawal

The withdrawal formulas are the inverse of the deposit formulas if no swap fees are considered. In other words, if you deposit a given amount of token _t_  for pool tokens and then immediately redeem these pool tokens for token _t_ again you should receive exactly what you started off with. 

The formula without considering swap fees is then:

$$
A_t = B_t \cdot \left(1-\left(1-\frac{P_{redeemed}}{P_{supply}}\right)^\frac{1}{W_t}\right)
$$

Where $$A_t$$ is the amount of token _t_  one receives when redeeming $$P_{redeemed}$$pool tokens.

Considering swap fees we have the following:

$$
A_t = B_t \cdot \left(1-\left(1-\frac{P_{redeemed}}{P_{supply}}\right)^\frac{1}{W_t}\right)\cdot \left(1-(1 - W_t)\cdot swapFee\right)
$$

If there were an exit fee it would be taken from the amount of tokens redeemed $$P_{redeemed}$$but as mentioned above this fee has been chosen to be zero in the first version of Balancer.

```text
/**********************************************************************************************
// _calc_SingleOutGivenPoolIn                                                                //
// tAo = tokenAmountOut            /      /                                             \\   //
// bO = tokenBalanceOut           /      // pS - (pAi * (1 - eF)) \     /    1    \      \\  //
// pAi = poolAmountIn            | bO - || ----------------------- | ^ | --------- | * b0 || //
// ps = poolSupply                \      \\          pS           /     \(wO / tW)/      //  //
// wI = tokenWeightIn      tAo =   \      \                                             //   //
// tW = totalWeight                    /     /      wO \       \                             //
// sF = swapFee                    *  | 1 - |  1 - ---- | * sF  |                            //
// eF = exitFee                        \     \      tW /       /                             //
**********************************************************************************************/
function _calc_SingleOutGivenPoolIn(
    uint tokenBalanceOut,
    uint tokenWeightOut,
    uint poolSupply,
    uint totalWeight,
    uint poolAmountIn,
    uint swapFee
)
    internal pure
    returns (uint tokenAmountOut)
{
    uint normalizedWeight = bdiv(tokenWeightOut, totalWeight);
    // charge exit fee on the pool token side
    // pAiAfterExitFee = pAi*(1-exitFee)
    uint poolAmountInAfterExitFee = bmul(poolAmountIn, bsub(BONE, EXIT_FEE));
    uint newPoolSupply = bsub(poolSupply,poolAmountInAfterExitFee);
    uint poolRatio = bdiv(newPoolSupply, poolSupply);

    // newBalTo = poolRatio^(1/weightTo) * balTo;
    uint tokenOutRatio = bpow(poolRatio, bdiv(BONE, normalizedWeight));
    uint newTokenBalanceOut = bmul(tokenOutRatio, tokenBalanceOut);

    uint tokenAmountOutBeforeSwapFee = bsub(tokenBalanceOut,newTokenBalanceOut);

    // charge swap fee on the output token side 
    //uint tAo = tAoBeforeSwapFee * (1 - (1-weightTo) * swapFee)
    uint zaz = bmul(bsub(BONE, normalizedWeight), swapFee); 
    tokenAmountOut = bmul(tokenAmountOutBeforeSwapFee, bsub(BONE, zaz));
    return tokenAmountOut;
}
```

Balancer also allows for a liquidity provider to choose a desired amount of token t, $$A_t$$, they would like to withdraw from the pool and calculates the necessary amount of pool tokens for that,$$P_{redeemed}$$. The formula without considering swap fees is:

$$
P_{redeemed} = P_{supply} \cdot \left(1-\left(1-\frac{A_t}{B_t}\right)^{W_t} \right)
$$

Where $$A_t$$ is the amount of token _t_  one receives when redeeming $$P_{redeemed}$$pool tokens.

Considering swap fees we have the following:

$$
P_{redeemed} = P_{supply} \cdot \left(1-\left(1-\frac{\frac{A_t}{\left(1-(1 - W_t)\cdot swapFee\right)}}{B_t}\right)^{W_t} \right)
$$

```text
/**********************************************************************************************
// _calc_PoolInGivenSingleOut                                                                //
// pAi = poolAmountIn               // /               tAo             \\     / wO \     \   //
// bO = tokenBalanceOut            // | bO - -------------------------- |\   | ---- |     \  //
// tAo = tokenAmountOut      pS - ||   \     1 - ((1 - (tO / tW)) * sF)/  | ^ \ tW /  * pS | //
// ps = poolSupply                 \\ -----------------------------------/                /  //
// wO = tokenWeightOut  pAi =       \\               bO                 /                /   //
// tW = totalWeight           -------------------------------------------------------------  //
// sF = swapFee                                        ( 1 - eF )                            //
// eF = exitFee                                                                              //
**********************************************************************************************/
function _calc_PoolInGivenSingleOut(
    uint tokenBalanceOut,
    uint tokenWeightOut,
    uint poolSupply,
    uint totalWeight,
    uint tokenAmountOut,
    uint swapFee
)
    internal pure
    returns (uint poolAmountIn)
{

    // charge swap fee on the output token side 
    uint normalizedWeight = bdiv(tokenWeightOut, totalWeight);
    //uint tAoBeforeSwapFee = tAo / (1 - (1-weightTo) * swapFee) ;
    uint zoo = bsub(BONE, normalizedWeight);
    uint zar = bmul(zoo, swapFee); 
    uint tokenAmountOutBeforeSwapFee = bdiv(tokenAmountOut, bsub(BONE, zar));

    uint newTokenBalanceOut = bsub(tokenBalanceOut,tokenAmountOutBeforeSwapFee);
    uint tokenOutRatio = bdiv(newTokenBalanceOut, tokenBalanceOut);

    //uint newPoolSupply = (ratioTo ^ weightTo) * poolSupply;
    uint poolRatio = bpow(tokenOutRatio, normalizedWeight);
    uint newPoolSupply = bmul(poolRatio, poolSupply);
    uint poolAmountInAfterExitFee = bsub(poolSupply,newPoolSupply);

    // charge exit fee on the pool token side
    // pAi = pAiAfterExitFee/(1-exitFee)
    poolAmountIn = bdiv(poolAmountInAfterExitFee, bsub(BONE, EXIT_FEE));
    return poolAmountIn;
}
```

