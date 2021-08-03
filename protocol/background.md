# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/background)

# Background

Automated Market Makers \(AMMs\) have been around in some form for as long as trades could be automated, starting in the traditional financial markets. AMMs are essentially automated agents, controlled by algorithms, that define rules for matching buyers and sellers to facilitate trades. Usually AMMs are continuously active in both directions of a trading pair. The liquidity provider's profit comes  from the spread between buy and sell prices.

Smart contract platforms like Ethereum have brought AMMs to a whole new level. They have, for the first time in history, combined the _trading algorithms_ with _custody_ of the underlying assets. This has led to interesting new features like atomic trading \(sometimes incorporating [flash loans](https://aave.com/flash-loans)\), instant feedback loops for correcting prices offered by an AMM, and more.

## AMMs in Ethereum

Alan Lu from [Gnosis](https://gnosis.io/) was the first in the Ethereum community to propose the simplest version of an AMM: a "pool," containing two tokens \(let's call them A and B\), in which the token prices are derived internally, using only the token balances, according to the simple invariant formula: balance\(A\) \* balance\(B\) = \(constant\) k, most commonly written as x\*y=k.

 Martin Köppelmann and Vitalik Buterin also promoted this idea, which Hayden Adams eventually implemented on [Uniswap](https://uniswap.org/).

![V. Buterin, &quot;On Path Independence&quot; \(linked below\)](../.gitbook/assets/xyk.png)

The "price curve," which can be steeper or gentler based on the choice of constant, defines the price of B in terms of A as the slope of the curve at the point defined by their relative balances. At any given time, we can represent the pool's current composition as a point on that curve.

The pool assets also have external market prices, so we can plot this "point" on the curve as well. If the external market prices diverge from the "internal" prices in either direction, these points will start to "move" away from each other along the curve. In response, arbitrageurs will buy or sell from the pool, trading against the market, changing the relative balances \(and therefore prices\), until the points converge again and balance is restored.

Balancer is essentially a generalization of the constant product rule to pools containing two or more tokens. In addition, Balancer Pools assign relative weights to each token, to accommodate pools of tokens with significantly different valuations. The weights represent the proportion of each token in the total pool. This flexibility greatly expands the utility of these pools, and allows for many interesting strategies and use cases, as we shall see.

## Further Reading

[Euler: The simplest exchange and currency](https://www.reddit.com/r/ethereum/comments/54l32y/euler_the_simplest_exchange_and_currency/)

[Decentralized Exchanges as Prediction Markets](https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way/)

[Building a Decentralized Exchange in Ethereum](https://blog.gnosis.pm/building-a-decentralized-exchange-in-ethereum-eea4e7452d6e) \(Gnosis / Alan Lu\)

[On Path Independence](https://vitalik.ca/general/2017/06/22/marketmakers.html) \(Vitalik Buterin\)

[Uniswap AMM](https://medium.com/scalar-capital/uniswap-a-unique-exchange-f4ef44f807bf) \(Medium article\)

[Improving front-running resistance of AMMs](https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers/1281)

[Advanced AMMs](https://www.cs.cmu.edu/~sandholm/automatedMarketMakersThatEnableNewSettings.AMMA-11.pdf) / [Practical AMMs](https://www.cs.cmu.edu/~./sandholm/liquidity-sensitive%20automated%20market%20maker.teac.pdf) \(CMU academic papers\)

