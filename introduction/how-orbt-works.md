---
description: '|| chain-agnostic, pre-funded, 1:1 liquid ||'
---

# How ORBT Works

ORBT’s architecture enables a seamless lifecycle of minting, deploying, and settling assets - ensuring every 0xAsset in circulation is verifiably backed and efficiently utilized.

<figure><img src="../.gitbook/assets/1.png" alt=""><figcaption></figcaption></figure>

### Example

{% stepper %}
{% step %}
[**Buy 0xAssets**](../user-flow-and-guides/how-to-buy-0xassets.md)

A user swaps \~100 USDT for **\~100** [**0xUSD**](../orbt-design/0xusd/) (minus minimal gas/AMM costs). Similarly, users can acquire [**0xBTC**](../orbt-design/0xassets.md#mechanics-and-policy-applies-to-0xbtc-0xeth) by swapping supported BTC-trackers into **0xBTC** at \~1:1.
{% endstep %}

{% step %}
**No spread on mint/redeem**

Any slippage or execution fees are **embedded in the swap/mint price**.
{% endstep %}

{% step %}
**Peg & reserves set by** [**UCE**](../orbt-design/uce/)

Keeps a liquid buffer for fast 1:1 [redemptions](../orbt-design/0xusd/redemption-rate-and-shortfall-socialization.md) and marks the remainder for monetization (pre-funded settlements / [money-market routing](../protocol-mechanics/integration/money-market-strategy.md)).
{% endstep %}

{% step %}
**Qualified custody for liquid funds**

The liquid backing (e.g., USDT) is held with **qualified custodians / safes** (BitGo, Fireblocks, Safe). Funds remain onchain/programmable and are available to honor **instant** [**PSM swaps**](../orbt-design/uce/concepts/swaps.md) and [redemptions](../orbt-design/0xusd/redemption-rate-and-shortfall-socialization.md) to keep[ 0xUSD](../orbt-design/0xusd/) at par, and routes the marked remainder fund into monetization.
{% endstep %}

{% step %}
**Pre-funded settlements**

ORBT sends **in-transit capital** through **policy governed monetization strategies** permits. Settlements execute, counterparties deliver, and funds are **returned** [**within 1–2 blocks**](../orbt-design/pocket/pocket-lifecycle-and-flows/intent-based-settlement-instant.md).
{% endstep %}

{% step %}
**Rewards to savers**

Pre-funded settlement routing margins, and money-market income are aggregated. Users who convert [0xAssets](../orbt-design/0xassets.md) into [**s0xAssets**](../user-flow-and-guides/how-to-buy-s0xasset.md) accrue this revenue as a [**savings rate**](../protocol-mechanics/savings-rate.md).
{% endstep %}
{% endstepper %}

<figure><img src="../.gitbook/assets/Frame 143724614.png" alt=""><figcaption></figcaption></figure>

***

### Pre-Funded Settlement

* **Liquid first:** A [**PSM reserve**](../orbt-design/uce/concepts/reserve-policy.md) sits ready to honor 1:1 swaps and redemptions so the peg holds under load.
* **Monetize the rest:** The **non-reserve** portion is pre-funded into settlement lanes where it earns **transaction fees**.
* **Fast round-trip:** Because settlements are pre-funded and [intent-based](intent-based-settlements.md), capital cycles back quickly, keeping liquidity **fresh** and peg pressure **low**.
* [**Policy controls**](../orbt-design/allocator/allocator-roles-and-responsibilities.md)**:** Per-asset limits, per-allocator caps, and daily ceilings are enforced by the UCE. Parameters can be tightened in stress to defend the peg.

<figure><img src="../.gitbook/assets/Frame 143724615 (1).png" alt=""><figcaption></figcaption></figure>

***

### 0xAsset variants

* [**0xUSD**](../orbt-design/0xusd/)**:** Dollar-par unit; base settlement currency across chains and protocols.
* **0xBTC:** BTC-par unit for BTC-denominated flows.
* **0xETH** (optional, if enabled): ETH-par unit for ETH-native settlements.

All [0xAssets](../orbt-design/0xassets.md) follow the same pattern: **1:1 issuance**, **PSM-based peg**, and **pre-funded monetization** for yield. They are chain-agnostic and composable.

***

### Savings assets (s0xAssets)

* [**s0xUSD**](../protocol-mechanics/savings-rate.md#how-it-works-s0xassets-your-savings-token)**, s0xBTC, …** are the **reward-accruing** versions of 0xAssets.
* [Deposit](../user-flow-and-guides/how-to-buy-0xassets.md) 0xAssets → [receive](../user-flow-and-guides/how-to-buy-s0xasset.md) s0xAssets → earn protocol revenue (allocator fees, routing margins, money-market spreads).
* [Redeem](../user-flow-and-guides/how-to-sell-s0xassets.md) anytime to withdraw your principal **plus accrued yield** (subject to any cooldowns/policy rules and jurisdictional limits).

***

### Custody & execution context

* **Qualified custodians / safes** (e.g., BitGo, Fireblocks, Safe) can hold reserves or operate [**Pocket** ](../orbt-design/pocket/)permits.
* [**Pocket** ](../orbt-design/pocket/)implements **least-privilege** policy keys on EOAs, multisigs, safes, custodial wallets, or smart contracts.
* The **Accounting Layer** (Money Market + Pocket) tracks credit delegation, limits, and settlement receipts end-to-end - **on-chain, auditable, and fast**.

***

<figure><img src="../.gitbook/assets/2.png" alt=""><figcaption></figcaption></figure>

#### Why this works

ORBT removes the “fragmented liquidity” tax. By unifying dollars into 0xAssets, keeping **high-quality liquid reserves**, and monetizing **in-transit capital** through pre-funded settlements, the system can&#x20;

(a) **defend the peg** and&#x20;

(b) **share real fee income**&#x20;

with savers - without exposing holders to anything except **on-chain execution and settlement flow risks** clearly described in the [Risk Disclosures](../resources/risk-disclaimer.md).
