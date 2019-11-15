# Math

The [Balancer whitepaper](https://balancer.finance/whitepaper.html) describes a set of formulas derived from the value function for interacting with the protocol. These functions represent the purest form of the math. The math used in the protocol differs in two key ways: exponentiation approximation and the addition of swap fees.

## Exponentiation Approximation

[Approxing](approxing.md)


### Spot Price

$$
SP^o_i = \frac{ \frac{B_i}{W_i} }{ \frac{B_o}{W_o} } 
$$

Where:

- $B_i$ is the balance of token *i*, the token being sold by the trader which is going *into* the pool.

- $B_o$ is the balance of token *o*, the token being bought by the trader which is going *out* of the pool.

- $W_i$ is the weight of token *i*

- $W_o$ is the weight of token *o*


When we consider swap fees, we do exactly the same calculations as without fees but using $A_i \cdot (1-swapFee)$ instead of only $A_i$. This strategy is referred to as charging fees "on the way in". With the swap fee, the spot price increases. It becomes then:

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
```


### Out-Given-In

$$
A_{o} = B_{o}  \cdot \left(1 - \left(\frac{B_{i}}{B_{i}+A_{i}}\right)^{\frac{W_{i}}{W_{o}}}\right)
$$

Using $A_i \cdot (1-swapFee)$ instead of $A_i$ we have the following formula:

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
```

### In-Given-Out


$$
A_{i} = B_{i} \cdot \left(\left(\frac{B_{o}}{B_{o}-A_{o}}\right)^{\frac{W_{o}}{W_{i}}}-1\right) 
$$


Since $A_i$ is the amount the user has to swap to get a desired amount out $A_o$, all we have to do to include fees is divide the formula above without fees by $(1-swapFee)$. This is because we know the fee charged on the way in will multiply that amount by $(1-swapFee)$. This will cross out both terms $(1-swapFee)$ and the amount out will be $A_o$ as desired:

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
```

### In-Given-Price

$$
A_i = B_i \cdot  \left( \left(\frac{SP'^o_i}{SP^o_i}\right)^\left({\frac{W_o}{W_o+W_i}}\right) - 1\right)
$$


Unfortunately, due to the non-linear characteristics of Balancer pools, when considering a swap fee there is no closed-form formula for $A_i$ given a desired spot price, say $SP1$. 

We propose an approximation to obtain $A_i$ such that the price is very close to the desired one, even in the presence of fees. 

An initial guess, $A_{i-e}$, is obtained by using the formula above without fees. By calculating the formula with $A_{i-e}$ we obtain $SP_e$. We then have an error that we would like to eliminate:

$Error = SP1 - SP_e$

To do that, we calculate an additional amount to be traded to get the pool spot price closer to the desired price $SP1$. Let's call that extra amount $A_{i-extra}$. 

We also need to define function $SpotPriceAfterSwap(A_i)$ which returns the spot price after a trade of $A_i$. 

The extra amount $A_{i-extra}$ is then calculated by dividing the price error by the derivative of function $SpotPriceAfterSwap(A_i)$ at $A_i = A_{i-e}$.

$$
A_{i-extra} = \frac{Error}{SpotPriceAfterSwap'(A_{i-e})}
$$

```text
    /**********************************************************************************************
    // _calc_InGivenPrice                                                                        //
    // _calc_InGivenPriceNoFee + extraAmountIn                                                   //
    **********************************************************************************************/
```

```text
    /**********************************************************************************************
    // _calc_InGivenPriceNoFee                                                                   //
    // aI  = tokenAmountIn                                                                       //
    // bI = tokenBalanceIn                    //   SP1    \     /   wO    \        \             //
    // SP0 = spotPriceBefore       aI = bI * || ---------  | ^ | --------  |   - 1  |            //
    // SP1 = spotPriceAfter                   \\   SP0    /     \ wO + wI /        /             //
    // wI = tokenWeightIn                                                                        //
    // wO = tokenWeightOut                                                                       //
    **********************************************************************************************/
```

```text
    /**********************************************************************************************
    // _calc_extraAmountIn                                                                       //
    // eAi = extraAmountIn               //                \      \                              //
    // aI = tokenAmountIn               || ( 1 - sF) * aI ) | + bI | * ( mP - SP1 )              //
    // mP = marginalPrice                \\                /      /                              //
    // bI = tokenBalanceIn      eAi =  -----------------------------------------------           //
    // wI = tokenWeightIn               /            /    wI \     (sF * bI) \                   //
    // wO = tokenWeightOut             | (1 - sF) * | 1 + --  | +  ---------  | * SP1            //
    // SP1 = spotPriceAfter             \            \    wO /     (aI + bI) /                   //
    // sF = swapFee                                                                              //
    **********************************************************************************************/
```


### All-Asset Deposit/Withdrawal

$$
D_k = \left(\frac{P_{supply}+P_{issued}}{P_{supply}}-1\right) \cdot B_k
$$

$$
A_k = \left(1-\frac{P_{supply}-P_{redeemed}}{P_{supply}}\right) \cdot B_k
$$

Balancer Labs takes a percentage of the pool tokens withdrawn as protocol fee. This is called the exit fee and is supposed to be around 1 to 10 basis points. From the amount of pool tokens the user redeems $P_{redeemed}$, Balancer Labs takes $exitFee \cdot P_{redeemed}$ pool tokens (that amount is not burned, it is just transferred to BLabs' multisig).