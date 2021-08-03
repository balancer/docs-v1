# Interact via Etherscan

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/withdrawing-liquidity-via-etherscan)

## Interact via Etherscan

Our UI allows for any liquidity provider to withdraw liquidity by clicking on the button "Remove Liquidity" on the detailed pool view page. This article though focus on how to withdraw liquidity without our UI \(in case it is down or inaccessible\). To do that we'll be using Etherscan.io.

If a pool is still private \(i.e. not finalized\), you can withdraw liquidity by simply unbinding each of its tokens:

![](https://lh5.googleusercontent.com/epkR6-l5a2awnjvRAjNtBkFqE2rjHe7gfZ5fo2BH0l1uj87IA3fN2n6mfhZEtIJ-VbrtHEhosjgo35k7sdEMhJhZSbyVz0AWeEoNCFn-YPG4fSRHtTznRaYSEGXQ_OM0KlsGOggo)

In case we’re dealing with a shared pool, things look a bit different. Once a pool is finalized, a BPT \(Balancer Pool Token\) associated with that pool is created with an initial supply of 100 units \(always with 18 decimals\). From that point on, ownership of the pool is tracked by BPT: holders are pro-rata owners of the pool’s liquidity.

Just to illustrate, this is how a BPT distribution looks like for the more popular “75-MKR / 25-WETH” pool:

[https://etherscan.io/token/tokenholderchart/0x987D7Cc04652710b74Fff380403f5c02f82e290a](https://etherscan.io/token/tokenholderchart/0x987D7Cc04652710b74Fff380403f5c02f82e290a)

![](https://lh5.googleusercontent.com/CwFnsvCn0E960qFS7qvBjXTSdVHgJyXoIW1skGhgvmelZrXJDTDCWVUh5GEY9AZlhd9r0vUy1_XGjb9vLp7ExH3xpVDkheUwT1yytFcyxJlUm_GBsj-q5L6s8XjCNSGKWanP5vuJ)

This means account 0x821a...46 owns about 25% of all the MKR and WETH held by the pool, and they may withdraw those tokens from the pool at any moment.

The simpler way to remove liquidity \(either fully or partially\) from a shared pool is of course to navigate to the pool’s page and click “Remove Liquidity”:

[https://pools.balancer.exchange/\#/pool/0x987D7Cc04652710b74Fff380403f5c02f82e290a](https://pools.balancer.exchange/#/pool/0x987D7Cc04652710b74Fff380403f5c02f82e290a)

### Bonus Points: Removing Liquidity from a Shared Pool via Etherscan

If you don’t want to depend on Balancer’s website being always operational; you may choose to run it locally \(it’s open-source\).

But maybe you want to understand a bit better what happens under the hood during a withdrawal.

So let’s use Etherscan to explore our [BTC++/DAI/USDC pool](https://etherscan.io/address/0x75286e183D923a5F52F52be205e358c5C9101b09#writeContract).

The function we’re looking for is called exitPool. The parameters expected are:

1. the amount of BPT \(in base units, so 1 is actually 10^18\) you’re giving back to the pool in exchange for the underlying tokens; and
2. the minimum amount of each underlying token \(also in base units, separated by commas\) you expect to receive \(transaction fails if any minimum isn’t met\).

To withdraw all your liquidity in one transaction, simply give back to the pool your full BPT balance \(in our case it’s a bit over 1184.12 BPT\). Remember all BPTs have 18 decimals. To accept any amount of underlying tokens, just set all minimum amounts to zero. Since we have 3 underlying tokens, minAmountsOut has 3 zeros: "0,0,0".

![](https://lh5.googleusercontent.com/T0sUcnFlu5-kXQmGbD2m8WAB5mgygcRblysrZVyGF1bTN4EYu9cW61GCSA0Sp2GTXC_gpThd1HejDGBMYar3flSdMcBYT8N55zNnqMOIwft32kn20aZykWD1zBW5JbqY3tKmhkO1)

And what is the point of getting to know the actual function being called and its parameters?

One use case that comes to mind is to keep handy \(e.g. saved on a mobile phone\) a pre-signed transaction withdrawing all your funds from a pool. This would be useful in an emergency: you could immediately broadcast your transaction, from any device \(that doesn’t hold the private key to that account\).

After clicking “Write” at the figure above, you can copy the DATA field shown in Metamask, and use it to craft an offline transaction using [MyEtherWallet](https://www.myetherwallet.com/interface/send-offline).

