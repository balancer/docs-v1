Above we describe two formulas which make use of exponentiation where both the base and exponent are fixed-point (non-integer) values:

\begin{equation}
\begin{gathered}
A_o = \left(1 - \left(\frac{B_i}{B_i+A_i}\right)^{\frac{W_i}{W_o}}\right).B_o
\end{gathered}
\end{equation}

\begin{equation}
\begin{gathered}
A_i = \left(\left(\frac{B_o}{B_o-A_o}\right)^{\frac{W_o}{W_i}}-1\right).B_i
\end{gathered}
\end{equation}


Since solidity does not have fixed point algebra or more complex functions like fractional power we use the following binomial approximation:

\begin{equation}
\begin{gathered}
\left(1+x\right)^\alpha=1+\alpha x+\frac{(\alpha)(\alpha-1)}{2!}x^2+
\frac{(\alpha)(\alpha-1)(\alpha-2)}{3!}x^3+
\cdots = \sum_{k=0}^{\infty}{\alpha \choose k}x^k
\end{gathered}
\end{equation}

which converges for ${|x| < 1}$
\\

\subsection{Derivation for $A_o$}
We derive the solidity approximation of the first equation using the binomial approximation above:

\begin{equation}
\begin{gathered}
A_o = \left(1 - \left(1+x\right)^{\alpha}\right).B_o
\end{gathered}
\end{equation}

where

\begin{equation}
\begin{gathered}
x = \frac{B_i}{B_i+A_i} - 1 = \frac{-A_i}{B_i+A_i}
\end{gathered}
\end{equation}

and

\begin{equation}
\begin{gathered}
\alpha = \frac{W_i}{W_o}
\end{gathered}
\end{equation}

$A_i$ is limited to at most $50\%$ of $B_i$. to ensure That is, no trade can exchange/sell more than $10\%$ of Balancer's balance of those tokens.

Since calculations in solidity are done in integers, the order of the operations we choose is fundamental for the calculation to be correct. E.g. $(99/100)*100 = 0$. This happens because $99/100$ is truncated to 0 and then multiplied by 100.
\\

To avoid dividing two numbers that are close to each other (which truncates all the precision as in the example above), we multiply by $B_o$ all terms in the binomial expansion used for approximating $A_o$:


\begin{equation}
\begin{gathered}
A_o=
\left(1 - \left(1+x\right)^{\alpha}\right).B_o =
\\
B_o - \left(B_o + B_o\alpha x+
B_o\frac{(\alpha)(\alpha-1)}{2!}x^2+
B_o\frac{(\alpha)(\alpha-1)(\alpha-2)}{3!}x^3+
\cdots\right) =
\\
-\left(B_o\alpha x+
B_o\frac{(\alpha)(\alpha-1)}{2!}x^2+
B_o\frac{(\alpha)(\alpha-1)(\alpha-2)}{3!}x^3+
\cdots \right)
\end{gathered}
\end{equation}



The binomial approximation described above is especially accurate for small values of $\alpha$. When $\alpha>1$ we split the calculation into two parts for increased accuracy:


\begin{equation}
\begin{gathered}
A_o = \left(1 - \left(\frac{B_i}{B_i+A_i}\right)^{int\left(\frac{W_i}{W_o}\right)}\left(\frac{B_i}{B_i+A_i}\right)^{\frac{W_i}{W_o}\%1}\right).B_o
\end{gathered}
\end{equation}

\subsection{Derivation for $A_i$}
Similarly to $A_o$, we have for $A_i$:

\begin{equation}
\begin{gathered}
A_i = \left(\left(1+x\right)^{\alpha}-1\right).B_i
\end{gathered}
\end{equation}

where

\begin{equation}
\begin{gathered}
x = \frac{B_o}{B_o-A_o} - 1 = \frac{A_o}{B_o-A_o}
\end{gathered}
\end{equation}

and

\begin{equation}
\begin{gathered}
\alpha = \frac{W_o}{W_i}
\end{gathered}
\end{equation}

To avoid dividing two numbers that are close to each other (which truncates all the precision as in the example above), we multiply by $B_i$ all terms in the binomial expansion used for approximating $A_i$:


\begin{equation}
\begin{gathered}
A_i=
\left(\left(1+x\right)^{\alpha}-1\right).B_i =
\\
\left(B_i + B_i\alpha x+
B_i\frac{(\alpha)(\alpha-1)}{2!}x^2+
B_i\frac{(\alpha)(\alpha-1)(\alpha-2)}{3!}x^3+
\cdots\right) - B_i =
\\
\left(B_i\alpha x+
B_i\frac{(\alpha)(\alpha-1)}{2!}x^2+
B_i\frac{(\alpha)(\alpha-1)(\alpha-2)}{3!}x^3+
\cdots \right)
\end{gathered}
\end{equation}

The binomial approximation described above is especially accurate for small values of $\alpha$. When $\alpha>1$ we split the calculation into two parts for increased accuracy:

$$
\begin{equation}
\begin{gathered}
A_i = \left(1 - \left(\frac{B_o}{B_o-A_o}\right)^{int\left(\frac{W_o}{W_i}\right)}\left(\frac{B_o}{B_o-A_o}\right)^{\frac{W_o}{W_i}\%1}\right).B_i
\end{gathered}
\end{equation}
$$


