# Testing on Kovan

## Overview

This guide will walk through the process of interacting with Balancer on Kovan using the front-end interfaces, directly with the smart contracts, and additional tooling such as the subgraph and SOR.

### Setup

This guide will use `seth` - a tool built by Dapphub to interact directly with smart contracts. To install, run the below command. Note: the Dapp Tools suite installs Nix OS 

```
curl https://dapp.tools/install | sh
```

{% hint style="info" %}
Nix is broken on MacOS Catalina. To fix, follow the steps at: [https://github.com/NixOS/nix/issues/2925\#issuecomment-539570232](https://github.com/NixOS/nix/issues/2925#issuecomment-539570232)
{% endhint %}

Follow the remaining steps at: [https://github.com/dapphub/dapptools/tree/master/src/seth\#example-sethrc-file](https://github.com/dapphub/dapptools/tree/master/src/seth#example-sethrc-file) in order to select the `SETH_CHAIN` and address / key.

Run the below lines in your terminal to setup environment variables that will be used later in the guide.

```text
BFACTORY=0x9c84391b443ea3a48788079a5f98e2ead55c9309
WETH=0xd0A1E359811322d97991E03f863a0C30C2cF029C
DAI=0xbaB6D2bf6C4cae495a958507AAf0804feCa2b000
MKR=0x8fD2D32dCde13F41BC7e6D966Edde0DfF1f92E2E
```

Acquire kovan ETH using the faucet at [https://faucet.kovan.network](https://faucet.kovan.network/) or request in the kovan eth gitter channel.

