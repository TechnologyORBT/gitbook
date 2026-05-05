---
description: '|| Built to Earn, Designed to Last ||'
---

# ORBT Overview

{% hint style="success" %}
[**0xAssets**](orbt-design/0xassets.md) **-** ORBT’s family of 1:1 tokens (e.g., [**0xUSD**](orbt-design/0xusd/), **0xBTC**, **0xETH**) - are **hard-pegged** to their native references (USD, BTC, ETH) and designed to be **always liquid and accessible across all chains**. They stay at par through deep, policy-controlled reserves, enabling **instant 1:1 swaps/redemptions** and powering **pre-funded,**[ **intent-based settlements**](introduction/intent-based-settlements.md) across chains and protocols.
{% endhint %}

### Overview

**ORBT is a** [**Unified Liquidity Layer (ULL)**](orbt-design/unified-liquidity-layer.md) **and modular stablecoin architecture** (on EVM) that provides a crypto native and scalable solution for **generating yield from pre-funded transactions**.

It mints liquid, 1:1 tokens called [**0xAssets**](orbt-design/0xassets.md) (e.g., 0xUSD, 0xBTC, 0xETH) that **solve fragmented liquidity** by letting value move simply **across any chain and settle anywhere** in the TradFi & DeFi ecosystem. [**0xAssets**](orbt-design/0xassets.md) are designed to be fully backed and easy to compose across DeFi. They stay at par through a tight mix of **liquid reserves** and **policy controls** so [redemptions](orbt-design/0xusd/redemption-rate-and-shortfall-socialization.md) are fast and predictable.

ORBT offers [**s0xAssets**](protocol-mechanics/savings-rate.md#how-it-works-s0xassets-your-savings-token) - the savings versions of 0xAssets. When users hold s0xAssets, the protocol routes the treasury into **pre-funded,**[ **intent-based settlements**](orbt-design/pocket/pocket-lifecycle-and-flows/intent-based-settlement-instant.md)**.** These are real transaction flows that earn [**allocator**](orbt-design/allocator/) **fees and routing margins**.&#x20;

ORBT aggregates that revenue and distributes it back to s0xAsset holders as native yield.

[**Peg stability**](orbt-design/0xusd/peg-mechanics-primary-and-secondary-paths.md) is supported by:

* **High liquid backing** to honor swaps and redemptions at 1:1.
* Liquid reserves that earn from native validation services and remain **100% liquid** at all times.
* **Monetization via pre-funded settlements**, which continuously refills protocol revenues and strengthens the peg over time.
* **On-chain transparency and fast access**, so liquidity returns to the pool within blocks defined by policy controlled allocations, keeping the system responsive under load.

{% hint style="info" %}
- Website: [https://orbt.xyz](https://orbt.xyz/)&#x20;
- Telegram: [https://t.me/ORBT\_Protocol ](https://t.me/ORBT_Protocol)
- Discord: [https://discord.gg/orbtprotocol](https://discord.gg/orbtprotocol)
- Twitter: [https://x.com/ORBT\_Protocol](https://x.com/ORBT_Protocol)
{% endhint %}

<details>

<summary><a href="user-flow-and-guides/how-to-buy-0xassets.md"><strong>How to Acquire 0xUSD</strong></a></summary>

Users currently have several options to acquire 0xUSD:

* **Permissionless Swap for 0xUSD:** Access external DEX or AMM pools to swap into or out of [0xUSD](orbt-design/0xusd/) using any supported assets (such as USDT or USDC). Once liquid markets are available, anyone can acquire 0xUSD by trading on supported exchanges without permission.
* **Direct Mint of 0xUSD:** Deposit accepted reserve assets (e.g. USDC, USDT or other approved collateral) into the ORBT Protocol to mint 0xUSD 1:1. \
  &#xNAN;_**Note:**_ Direct minting may be subject to certain criteria or whitelisting (e.g. KYC/KYB checks for approved market-making counterparties), as outlined in the supplemental 0xUSD [Terms and Conditions](resources/0xassets-terms-and-conditions.md).
* **Redemption of 0xUSD:** 0xUSD can be swapped on DEXs with any supported assets or via the [Unified Collateral Engine(UCE)](orbt-design/uce/), or redeemed via the UCE for the underlying assets from the protocol’s reserves; this mechanism allows converting 0xUSD back into base assets (like USDC/USDT) at minimal slippage.

</details>

<details>

<summary><a href="user-flow-and-guides/how-to-buy-s0xasset.md"><strong>How to save using 0xUSD</strong></a></summary>

One of ORBT’s core advantages is enabling users to earn a **sustainable yield** on their dollar assets. Here’s how you can earn savings rate on 0xUSD (based on current blended strategy returns):

* [**Buy s0xUSD**](user-flow-and-guides/how-to-buy-s0xasset.md)**:** By **supplying** your 0xUSD in the ORBT savings module and holding **s0xUSD**, you gain access to the protocol’s curated strategies. The yield from ORBT’s Pre-Funded Settlements deployments is distributed to stakers as rewards, creating a steady savings rate. Your staked 0xUSD remains fully backed and continues to track the U.S. dollar, accruing rewards over time.&#x20;
* [**Reedem s0xUSD**](user-flow-and-guides/how-to-sell-s0xassets.md)**:** You can **reedem** at any time to withdraw your 0xUSD **plus the accumulated yield.**&#x20;

By combining a stable USD peg with transparent, diversified yield generation, ORBT aims to offer a crypto-native dollar that not only **holds its value** but also **works for you** – growing over time through safe, scalable strategies. With 0xUSD, users no longer have to choose between stability and yield; the ORBT protocol delivers both in a single, composable asset designed for the future of decentralized finance.

</details>
