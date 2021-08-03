# Experimental

## This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/guides/smart-pool-templates-gui/experimental)

## Experimental

If you've been in the crypto space since 2017, you'll remember well the ICO hype. People were throwing money at every new token with a website. \(If you're just a bit older, you'll remember the .com boom of '99 as well - when it was every company with a website.\)

![.COM -&amp;gt; ICO -&amp;gt; DeFi](../../.gitbook/assets/takemoney.jpeg)

We are now seeing the same phenomenon in DeFi. Cross out "token," insert "protocol." A high school student at a hackathon can deploy a contract written over a sleepless weekend - and by Monday there's $7 million dollars locked in it!

This is dangerous, to say the least. Even worse, with everything public and open source, even if you don't intend to release something - [literally anyone](https://cointelegraph.com/news/anonymous-developer-deploys-curve-contracts-forcing-early-launch) can copy and deploy it for you.

Capped pools are a way to - if not completely eliminate the danger - at least limit the potential damage. Furthermore, it is flexible enough to allow the pool controller to halt and resume the influx of new capital at any time, for any reason.

Legend**: required;** ~~not required;~~ _optional_

**Rights configuration:**

* _canPauseSwapping_
* _canChangeSwapFee_
* ~~canChangeWeights~~
* ~~canAddRemoveTokens~~
* ~~canWhitelistLPs~~
* **canChangeCap**

Change cap is required, of course. You might also want to be able to control \(or at least influence\) trading through pausing or setting fees, but that is optional.

The "cap" refers to the total supply of pool tokens. Recall that pool tokens are themselves ERC-20s, so this corresponds to the `totalSupply()`.

The cap is always there, but if the right is not enabled, it is always set to `MAX` \(i.e., unlimited\). If the right is enabled, the cap is set to the initial supply on `createPool()`. Immediately after creating the pool, no one can add liquidity \(including the controller\).

To allow others to join the pool, the controller can set the cap higher - for instance, to a supply corresponding to a given total underlying asset value. At any point, they can raise or lower the cap - including to 0 \(to "freeze" all new investment\), or unlimited \(if confidence is high and it's party time\).

