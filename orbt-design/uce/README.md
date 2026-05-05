---
description: Unified Collateral Engine
---

# UCE

The **Unified Collateral Engine (UCE)** is the backbone of **0xAsset** issuance and solvency. It aggregates and manages collateral deposits that back the protocol’s stablecoin (**0xUSD**) and synthetic assets (**0xBTC**, **0xETH**). Unlike legacy systems that siloed collateral by asset type or vault, the UCE operates through a **unified collateral pool** governed by a consistent risk framework.

{% hint style="info" %}
_**Unified Pool Advantage**_: The unified collateral pool maximizes capital efficiency. Collateral surpluses from one asset type can offset deficiencies in another, reducing under-collateralization risk and minimizing idle capital.
{% endhint %}

The UCE maintains a hard peg of 1:1 using PSM and **on-chain oracles** reassuring the peg for all minted 0xAssets.&#x20;

This unified design increases capital efficiency by allowing diverse collateral assets to secure the same stablecoin / synthetic assets and enabling surplus value in one area to offset shortfalls in another — all within governance-defined risk limits.

Conceptually, the UCE smart contract design philosophy is heavily inspired from **Sky & Spark's Dss-LitePSM and PSM3** respectively. However, ORBT's implementation scales this model by supporting more than one correlated collateral assets enhancing the peg management of the synths even better, along with managing cross-chain collateralization and integration with unified liquidity and modular yield layer.&#x20;

{% hint style="warning" %}
_**Cross-Chain Collateral Risk**_**:** Cross-chain collateral introduces latency and bridge dependencies. Governance may apply additional risk premiums or constraints to assets sourced from external chains to mitigate exposure to bridge or oracle failures.
{% endhint %}

All state changes within the UCE are **fully transparent and auditable on-chain**. Risk parameters — including collateral types, ratios, and liquidation thresholds — are configured and maintained through **ORBT governance**, ensuring system solvency and long-term stability.

The Unified Collateral Engine underpins ORBT’s solvency model, enabling efficient, secure, and auditable collateral management across multiple chains and asset types — forming the foundation for 0xUSD and all synthetic 0xAssets.

