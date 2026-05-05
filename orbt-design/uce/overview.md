# Overview

The **Unified Collateral Engine (UCE)** is the core liquidity and asset management layer of the **ORBT protocol**. It functions as a sophisticated **Peg Stability Module (PSM)** that enables seamless, low-slippage swaps between underlying collateral assets (e.g., _WBTC, cbBTC_) and their corresponding synthetic [**0xAssets**](../0xassets.md) (e.g., _0xBTC_).\
It serves as the **primary liquidity hub** through which all participants - users, allocators, and integrators - interact with the ORBT ecosystem.

Unlike traditional systems that only focus on holding collateral, the UCE introduces a **dynamic credit and liquidity framework**. Whitelisted [**allocators**](concepts/allocators/) can access credit lines, mint 0xAssets, and channel liquidity into strategies or money markets - creating a continuous, self-balancing flow of capital across the protocol.

### Purpose & Role in the Ecosystem

The UCE is designed to be the **heartbeat of ORBT’s liquidity network**.\
It ensures that every 0xAsset in circulation is:

* **Fully backed** by verified collateral,
* **Instantly redeemable** through deep on-chain liquidity, and
* **Efficiently utilized** through yield-bearing integrations.

By combining stable asset management, yield generation, and credit provisioning, the UCE bridges traditional PSM mechanics with next-generation DeFi liquidity design - offering **instant settlement** and **capital productivity** in a single framework.

### How it works

{% stepper %}
{% step %}
#### **Collateral Inflow**

Users or allocators [deposit supported collateral](../../user-flow-and-guides/how-to-buy-0xassets.md) (e.g., USDC, WBTC) into the UCE.
{% endstep %}

{% step %}
#### **Minting Synthetic Assets:**

Based on collateral ratios or credit limits, the [UCE issues 0xAssets](concepts/swaps.md) (like 0xUSD or 0xBTC) directly to PSM allowing users to swap the assets directly
{% endstep %}

{% step %}
#### **Active Liquidity Management**

All the underlying assets are deployed into [**money markets**](../orbt-money-market-liquidity-custodian/) or **vaults** to earn yield while maintaining instant redemption ratios.
{% endstep %}

{% step %}
#### **Redemptions & Peg Stability**

When users redeem, the UCE settles instantly from its [on-hand reserves](concepts/reserve-policy.md). If reserves dip, it dynamically pulls liquidity from connected pockets - ensuring peg consistency without delays.
{% endstep %}

{% step %}
#### **Governance & Adjustments**

Parameters like reserve ratios, fee curves, and credit ceilings are continuously optimized through on-chain governance.
{% endstep %}
{% endstepper %}
