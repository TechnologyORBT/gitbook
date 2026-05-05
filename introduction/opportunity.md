---
description: '|| where pre-funded fees become user yield ||'
---

# Opportunity

### Market size & the leaking fee pool

{% columns %}
{% column %}
<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
{% endcolumn %}

{% column %}
Liquidity today is routed across chains, AMMs, CEXs, bridges and custodians. Movement still happens via **pre-funded settlements**, and every hop charge slippage and venue fees that don’t accrue to the asset holder.&#x20;

* **On-chain swaps \~$1.8T/yr** at **0.25–0.75%**
* **Global remittances \~$900B/yr** at **1–5%**.&#x20;

That implies an existing fee pool of roughly **$13.5B (low case)** to **$58.5B (high case)** paid annually just to access liquidity.&#x20;

None of that is unified, and almost none is shared back to the users who fund it.
{% endcolumn %}
{% endcolumns %}

***

### Why this is monetizable now

Three ingredients make this capturable without inventing new risk primitives:

1. [**0xAssets** ](../orbt-design/0xassets.md)**unify liquidity** into a single, chain-agnostic reserve with instant exits via [**UCE**](../orbt-design/uce/)
2. **Pre-funded, intent-based settlements** route the _non-reserve_ portion through trusted OCHs using [**Pocket** ](../orbt-design/pocket/)permits, earning real transaction fees (incl. allocator **5% flat** + routing margins) and returning funds **within 1–2 blocks**.
3. [**Policy controls**](../orbt-design/uce/concepts/reserve-policy.md) enforce per-asset reserves, [per-allocator caps](../orbt-design/allocator/allocator-roles-and-responsibilities.md) and daily ceilings so peg integrity and liquidity freshness are maintained while the system systematically taps recurring settlement revenues.

***

### Who benefits as this scales

* **Users/Treasuries:** Hold [**0xUSD**](../user-flow-and-guides/how-to-buy-0xassets.md) for clean 1:1 liquidity; deposit for [**s0xUSD**](../user-flow-and-guides/how-to-buy-s0xasset.md) to share in the fees you already pay across swaps and remittances.
* **Protocols & integrators:** [Plug into 0xAssets](../protocol-mechanics/integration/) for instant settlement liquidity without pre-funding a dozen venues.
* [**Allocators**](../orbt-design/allocator/)**/OCHs:** Clear more volume under transparent policy; earn their 5% fee plus profit-share while reinforcing peg liquidity.

**Context note:** ORBT’s early infrastructure has already processed **$10.5B+** in settlement flows with **$20B+** pre-funded transactions and millions of intents cleared - demonstrating operational readiness for institutional throughput and fast capital return.

<figure><img src="../.gitbook/assets/section-cumulative-rewards.png" alt=""><figcaption></figcaption></figure>

***

### Why this isn’t “just another stablecoin pitch”

{% hint style="success" %}
Existing stablecoins are excellent units of account but **don’t unify liquidity** and **don’t return the fee stream** to holders. Emissions-driven yields decay; AMM yields are path-dependent and expose LPs to IL. A **Unified Liquidity Layer** explicitly converts **pre-funded settlement costs** into **predictable, policy-controlled savings yield** while preserving instant redemptions and on-chain transparency.
{% endhint %}
