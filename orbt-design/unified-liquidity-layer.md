---
description: '|| Universal Liquidity Backbone ||'
---

# Unified Liquidity Layer

**ORBT** introduces the **Unified Liquidity Layer (ULL)**, a foundational liquidity framework that standardizes asset movement and yield generation across the EVM ecosystem.\
The ULL serves as a unifying base layer where assets, liquidity, and cross-chain interactions converge seamlessly, removing the friction caused by fragmented liquidity pools and isolated markets.

Through the ULL, ORBT enables users, protocols, and institutions to interact with a consistent liquidity standard known as [**0xAssets**](0xassets.md), designed for interoperability, scalability, and efficiency.

***

### Why a Unified Liquidity Layer?

DeFi liquidity today is scattered. The same asset can exist as multiple wrapped versions such as `BTC.b`, `WBTC`, `tBTC`, or `BTC.e`, each siloed within its own chain or protocol.\
This fragmentation creates inefficiencies in yield, interoperability, and capital deployment.

The **Unified Liquidity Layer** solves this by turning liquidity into a [single, composable substrate](0xassets.md) that exists across chains.\
Every asset under ORBT becomes part of a unified ecosystem that is mobile, yield-generating, and natively composable.

***

### 0xAssets: Unified Representation of Liquidity

At the core of the ULL lies the [**0xAsset standard**](0xassets.md), an interoperable family of tokens that represent real assets across different chains.

Each 0xAsset maintains a 1:1 relationship with its [underlying base asset](uce/concepts/swaps.md#instrument-classes) but exists as a native unit of liquidity within the ORBT ecosystem.

#### **Example**

* A user deposits **ETH** on **Ethereum Mainnet**.
* ORBT mints **0xETH**, the unified version of ETH within the ULL.
* This **0xETH** can be:
  * Staked in yield strategies
  * Used as collateral in lending protocols
  * Bridged natively to other chains such as Arbitrum, Base, or Polygon

<mark style="background-color:yellow;">**No wrappers. No synthetic tokens.**</mark>\
Just one liquidity standard recognized across all integrated environments.

#### **Benefits**

* Solves asset fragmentation
* Enables consistent yield strategies across chains
* Provides composable liquidity for any EVM-compatible protocol

### **Cross-Chain Mobility: Native Bridging**

The ULL also functions as a **native bridge** for 0xAssets.\
Users can move liquidity across supported chains directly through the ORBT protocol without depending on third-party bridges or wrapped representations.

#### **Real-World Example**

Suppose Alice has **0xBTC** on **Base** and wants to deploy it on **Polygon** to participate in a savings pool.

1. Alice initiates a bridge transfer via ORBT.
2. ORBT’s liquidity registry verifies the asset and burns her **0xBTC(Base)**.
3. Simultaneously, ORBT releases **0xBTC(Polygon)** to her same wallet address.

The process is instantaneous, secured by ORBT’s cross-chain verification and pre-funded liquidity mechanism.

This native movement makes ORBT a cross-chain liquidity router rather than just a token bridge.

***

Every transaction within the ULL, whether bridging, swapping, or reallocating, leverages **pre-funded liquidity pools**.\
These pools are designed to provide **instant settlement** while generating yield on the locked capital.

When users bridge 0xUSDC between chains, their transaction uses liquidity that is already pre-deployed by ORBT’s vaults.\
That idle liquidity is simultaneously earning yield through integrated strategies, ensuring the network’s liquidity remains productive, not static.

Thus, every movement of capital contributes to both **user utility** and **ecosystem yield**.
