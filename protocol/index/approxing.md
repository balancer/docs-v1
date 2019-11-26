# Exponentiation

Above we describe two formulas which make use of exponentiation where both the base and exponent are fixed-point \(non-integer\) values:

\begin{equation} \begin{gathered} A\_o = \left\(1 - \left\(\frac{B\_i}{B\_i+A\_i}\right\)^{\frac{W\_i}{W\_o}}\right\).B\_o \end{gathered} \end{equation}

\begin{equation} \begin{gathered} A\_i = \left\(\left\(\frac{B\_o}{B\_o-A\_o}\right\)^{\frac{W\_o}{W\_i}}-1\right\).B\_i \end{gathered} \end{equation}

Since solidity does not have fixed point algebra or more complex functions like fractional power we use the following binomial approximation:

\begin{equation} \begin{gathered} \left\(1+x\right\)^\alpha=1+\alpha x+\frac{\(\alpha\)\(\alpha-1\)}{2!}x^2+ \frac{\(\alpha\)\(\alpha-1\)\(\alpha-2\)}{3!}x^3+ \cdots = \sum\_{k=0}^{\infty}{\alpha \choose k}x^k \end{gathered} \end{equation}

which converges for ${\|x\| &lt; 1}$ \

\subsection{Derivation for $A\_o$} We derive the solidity approximation of the first equation using the binomial approximation above:

\begin{equation} \begin{gathered} A\_o = \left\(1 - \left\(1+x\right\)^{\alpha}\right\).B\_o \end{gathered} \end{equation}

where

\begin{equation} \begin{gathered} x = \frac{B\_i}{B\_i+A\_i} - 1 = \frac{-A\_i}{B\_i+A\_i} \end{gathered} \end{equation}

and

\begin{equation} \begin{gathered} \alpha = \frac{W\_i}{W\_o} \end{gathered} \end{equation}

$A\_i$ is limited to at most $50\%$ of $B\_i$. to ensure That is, no trade can exchange/sell more than $10\%$ of Balancer's balance of those tokens.

Since calculations in solidity are done in integers, the order of the operations we choose is fundamental for the calculation to be correct. E.g. $\(99/100\)\*100 = 0$. This happens because $99/100$ is truncated to 0 and then multiplied by 100. \

To avoid dividing two numbers that are close to each other \(which truncates all the precision as in the example above\), we multiply by $B\_o$ all terms in the binomial expansion used for approximating $A\_o$:

\begin{equation} \begin{gathered} A\_o= \left\(1 - \left\(1+x\right\)^{\alpha}\right\).B\_o = \ B\_o - \left\(B\_o + B\_o\alpha x+ B\_o\frac{\(\alpha\)\(\alpha-1\)}{2!}x^2+ B\_o\frac{\(\alpha\)\(\alpha-1\)\(\alpha-2\)}{3!}x^3+ \cdots\right\) = \ -\left\(B\_o\alpha x+ B\_o\frac{\(\alpha\)\(\alpha-1\)}{2!}x^2+ B\_o\frac{\(\alpha\)\(\alpha-1\)\(\alpha-2\)}{3!}x^3+ \cdots \right\) \end{gathered} \end{equation}

The binomial approximation described above is especially accurate for small values of $\alpha$. When $\alpha&gt;1$ we split the calculation into two parts for increased accuracy:

\begin{equation} \begin{gathered} A\_o = \left\(1 - \left\(\frac{B\_i}{B\_i+A\_i}\right\)^{int\left\(\frac{W\_i}{W\_o}\right\)}\left\(\frac{B\_i}{B\_i+A\_i}\right\)^{\frac{W\_i}{W\_o}\%1}\right\).B\_o \end{gathered} \end{equation}

\subsection{Derivation for $A\_i$} Similarly to $A\_o$, we have for $A\_i$:

\begin{equation} \begin{gathered} A\_i = \left\(\left\(1+x\right\)^{\alpha}-1\right\).B\_i \end{gathered} \end{equation}

where

\begin{equation} \begin{gathered} x = \frac{B\_o}{B\_o-A\_o} - 1 = \frac{A\_o}{B\_o-A\_o} \end{gathered} \end{equation}

and

\begin{equation} \begin{gathered} \alpha = \frac{W\_o}{W\_i} \end{gathered} \end{equation}

To avoid dividing two numbers that are close to each other \(which truncates all the precision as in the example above\), we multiply by $B\_i$ all terms in the binomial expansion used for approximating $A\_i$:

\begin{equation} \begin{gathered} A\_i= \left\(\left\(1+x\right\)^{\alpha}-1\right\).B\_i = \ \left\(B\_i + B\_i\alpha x+ B\_i\frac{\(\alpha\)\(\alpha-1\)}{2!}x^2+ B\_i\frac{\(\alpha\)\(\alpha-1\)\(\alpha-2\)}{3!}x^3+ \cdots\right\) - B\_i = \ \left\(B\_i\alpha x+ B\_i\frac{\(\alpha\)\(\alpha-1\)}{2!}x^2+ B\_i\frac{\(\alpha\)\(\alpha-1\)\(\alpha-2\)}{3!}x^3+ \cdots \right\) \end{gathered} \end{equation}

The binomial approximation described above is especially accurate for small values of $\alpha$. When $\alpha&gt;1$ we split the calculation into two parts for increased accuracy:

$$
\begin{equation}
\begin{gathered}
A_i = \left(1 - \left(\frac{B_o}{B_o-A_o}\right)^{int\left(\frac{W_o}{W_i}\right)}\left(\frac{B_o}{B_o-A_o}\right)^{\frac{W_o}{W_i}\%1}\right).B_i
\end{gathered}
\end{equation}
$$

