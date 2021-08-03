# Exponentiation

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/core-concepts/protocol/index/approxing)

## Exponentiation

The main formulas used in Balancer protocol make use of a form of exponentiation where both the base and exponent are fixed-point \(non-integer\) values. Take for example the `swap` functions, where the weights in both the exponent and the base are fractions:

$$
A_o = \left(1 - \left(\frac{B_i}{B_i+A_i}\right)^{\frac{W_i}{W_o}}\right).B_o
$$

$$
\begin{equation} \begin{gathered} A_i = \left(\left(\frac{B_o}{B_o-A_o}\right)^{\frac{W_o}{W_i}}-1\right).B_i \end{gathered} \end{equation}
$$

Since solidity does not have fixed point algebra or more complex functions like fractional power we use the following binomial approximation:

$$
\begin{equation} \begin{gathered} \left(1+x\right)^\alpha=1+\alpha x+\frac{(\alpha)(\alpha-1)}{2!}x^2+ \frac{(\alpha)(\alpha-1)(\alpha-2)}{3!}x^3+ \cdots = \sum_{k=0}^{\infty}{\alpha \choose k}x^k \end{gathered} \end{equation}
$$

which converges for $${|x| < 1}$$.

When $$\alpha>1$$ we split the calculation into two parts for increased accuracy, the first is the exponential with the integer part of $$\alpha$$ \(which we can calculate exactly\) and the second is the exponential with the fractional part of $$\alpha$$:

$$
\begin{equation}
\begin{gathered}
A_i = \left(1 - \left(\frac{B_o}{B_o-A_o}\right)^{int\left(\frac{W_o}{W_i}\right)}\left(\frac{B_o}{B_o-A_o}\right)^{\frac{W_o}{W_i}\%1}\right).B_i
\end{gathered}
\end{equation}
$$

