# Background

Automated Market Makers (AMMs) have been around in their simplest form for as long as trades could be automated in the conventional financial markets. AMMs are essentially automated agents that are controlled by algorithms that define rules for trades. Usually AMMs are continuously active in both directions of a trading pair. It's profit comes basically from a spread between buy and sell prices. 

Smart contract platforms like Ethereum have brought AMMs to a whole new level. They have, for the first time in history, combined both the *custody* of assets traded by AMMs and also the *algorithms* dictating how the assets should be traded. This led to interesting new features like atomic trading, instant feedback loops for correcting prices offered by an AMM among others.

## AMMs in Ethereum

Alan Lu from Gnosis has been the first in the Ethereum community to propose the simplest version of AMM: a pool with two tokens that respects the constant product formula k = x * y, meaning that the multiplication of the token balances is a constant. Martin Köppelmann and Vitalik Buterin also promoted this idea that eventually got implemented by Hayden Adams on Uniswap.

Balancer can be seen as a generalization of the constant product rule, allowing for 2 or more tokens as well as the choice of arbitrary value weights for each token. The weigths represent the share of value each token represents in the total pool value.

## Further Reading

[https://www.reddit.com/r/ethereum/comments/54l32y/euler\_the\_simplest\_exchange\_and\_currency/](https://www.reddit.com/r/ethereum/comments/54l32y/euler_the_simplest_exchange_and_currency/)

[https://www.reddit.com/r/ethereum/comments/55m04x/lets\_run\_onchain\_decentralized\_exchanges\_the\_way/](https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way/)

[https://blog.gnosis.pm/building-a-decentralized-exchange-in-ethereum-eea4e7452d6e](https://blog.gnosis.pm/building-a-decentralized-exchange-in-ethereum-eea4e7452d6e)

[https://vitalik.ca/general/2017/06/22/marketmakers.html](https://vitalik.ca/general/2017/06/22/marketmakers.html)

[https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers/1281](https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers/1281)

[https://www.cs.cmu.edu/~sandholm/automatedMarketMakersThatEnableNewSettings.AMMA-11.pdf](https://www.cs.cmu.edu/~sandholm/automatedMarketMakersThatEnableNewSettings.AMMA-11.pdf)

[https://www.cs.cmu.edu/~./sandholm/liquidity-sensitive automated market maker.teac.pdf](https://www.cs.cmu.edu/~./sandholm/liquidity-sensitive%20automated%20market%20maker.teac.pdf)

[https://medium.com/scalar-capital/uniswap-a-unique-exchange-f4ef44f807bf](https://medium.com/scalar-capital/uniswap-a-unique-exchange-f4ef44f807bf)

