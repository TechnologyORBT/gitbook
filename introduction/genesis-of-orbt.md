# Genesis of ORBT

## A Protocol Born from Infrastructure Gaps in On-Chain Finance

Over the last 2 years, decentralized finance reached scale but not maturity. Billions continue to flow through stablecoins, money markets, and synthetic assets, yet the infrastructure to **settle institutional transactions across counterparties, intents, and networks** remains fragmented and inefficient. Stablecoins provide liquidity, and money markets offer yield, but institutions lack a **universal layer to clear and settle “what I want” into “what I have”** with speed, predictability, and real economics. Atomic swaps create **liquidity silos**, while most yield models rely on emissions and speculation rather than revenue and usage.

As founders who have built real-world businesses and invested early in blockchain and DeFi, we lived this gap firsthand. We were familiar with the existing models, and skeptical of their sustainability. What DeFi needed was **business-driven, transparent yield,** tied to verifiable transaction activity and **institutional-grade settlement**.

## Observing the Gap: A Missing Settlement Fabric and Lack of Predictable and Sustainable Yield

Despite billions in on-chain liquidity, decentralized finance still lacks the infrastructure to settle institutional transactions with predictability, efficiency, and yield integrity. The founding team, with backgrounds in DeFi infrastructure, liquidity provisioning, and treasury management, recognized three interlocking structural gaps that constrain the liquidity markets:

{% stepper %}
{% step %}
### Fragmented and Inefficient Intent-Based Settlement for Institutions

Most protocols still require users to bring both sides of a swap at once (e.g., _ETH → USDC_), offering no mechanism to express or fulfill delayed, cross-chain, or scheduled intents. What if you simply wanted to express a need “I want to pay X in 30 minutes” and leave the fulfillment to an efficient market layer? The DeFi world lacks a clearing system that can batch and fulfill intent-based swaps and payments, especially across chains or with delayed execution.

Institutions, treasuries, and enterprise participants need a universal clearing layer that can batch, route, and execute intent-based payments with guaranteed finality and predictable economics. Without this, liquidity remains siloed across chains and AMMs, creating shallow pools, high slippage, impermanent loss scenarios and low capital efficiency.
{% endstep %}

{% step %}
### Idle Liquidity and Unproductive Capital

Even in active protocols, billions in idle assets sit dormant in vaults or pools. Worse, the deployed capital often earns opaque, unstable returns driven by speculative emissions or leverage, not by real transaction flow. This structural inefficiency leaves trillions in global liquidity unutilized and locked in treasuries, exchanges, and custodial networks that cannot access yield linked to verifiable economic activity.

The team envisioned a segregated collateral and settlement engine that keeps assets liquid while earning transparent, sustainable yield from money-market interest, swap fees, and settlement revenue.
{% endstep %}

{% step %}
### The Yield Crisis: Unstable, Speculative and Unsustainable

For years, DeFi yields have been artificially inflated by token emissions; an unsustainable cycle of short-term incentives rather than durable cash flows. Institutional allocators remain sidelined due to:

* Unstable returns from impermanent loss and venue risk.
* Opaque routing and limited auditability.
* Lack of risk-adjusted yield benchmarks tied to actual economic throughput.

ORBT set out to redefine this foundation: to deliver predictable, sustainable yield directly derived from protocol fees, interest income, and the native yield of underlying assets, targeting a decent savings rate without speculative farming or inflationary models.
{% endstep %}
{% endstepper %}

## Engineering a New Layer: Collateral + Clearing + Yield

The first commit in the ORBT [GitHub repo](https://github.com/Orbtofficial) wasn’t about speculation - it was about infrastructure. The team architected a protocol that combined:

* [**Unified Collateral Engine (UCE)**](../orbt-design/uce/)**:** A smart contract that accepts whitelisted assets (e.g., USDC, BTC) and splits liquidity into reserve and yield-seeking strategies.
* [**0xAssets**](../orbt-design/0xassets.md) **(e.g.,** [**0xUSD**](../orbt-design/0xusd/)**, 0xBTC):** Fully collateralized, yield-bearing synthetic representations of major assets, backed 1:1 and redeemable on-demand.
* [**User Savings Module (USM) Vaults**](../orbt-design/usm/)**:** ERC-4626 vaults that allow users to deposit [0xAssets](../orbt-design/0xassets.md) and earn sustainable APY through protocol revenue and money market integration.
* [**Pockets**](../orbt-design/pocket/) **&** [**Allocators**](../orbt-design/allocator/)**:** Smart contract-controlled addresses that hold segregated liquidity, deploy into [Aave](../orbt-design/pocket/aave-only-pocket-strategy/) and other markets, and execute [intent-based payments](../orbt-design/pocket/pocket-lifecycle-and-flows/intent-based-settlement-instant.md) on behalf of users.
* [**Intent Settlement Layer**](intent-based-settlements.md)**:** A solver-filler architecture allowing instant execution of batched user intents, such as FX conversions or payouts, with liquidity provided from pre-funded or credit-based vaults.

The protocol is designed to charge small, dynamic fees (e.g., _10–50 bps_) on settlement volume, earn money market interest, and capture spread via market-based fulfillment. These revenues are funneled back into vaults and savings contracts, growing the exchange rate of [**s0xAssets**](../orbt-design/usm/) over time.

## Target Outcome

ORBT is building the standard yield layer for digital assets, where liquidity earns directly from intent based settlements. By combining proven settlement economics with cross-chain interoperability through its [**Unified Liquidity Layer**](../orbt-design/unified-liquidity-layer.md) **&** [**0xAsset**](../orbt-design/0xassets.md) framework, ORBT delivers predictable, scalable, and institutional-grade yield that can support billions in capital and redefine how liquidity earns in decentralized finance.
