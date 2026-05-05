# Foundational Principles

ORBT’s design is guided by a clear set of **foundational principles** that balance decentralization, scalability, and real-world utility.\
At its core, ORBT aims to unify liquidity across chains and user types - from permissionless DeFi participants to regulated institutional players - without compromising on transparency or trustlessness.

### 1. Unified Liquidity, Modular Execution

ORBT’s most defining goal is to **aggregate fragmented liquidity** into a single interoperable framework.\
Instead of siloed pools per chain or per strategy, ORBT’s architecture routes capital dynamically through [**Pocket** ](../pocket/)**contracts** and the [**ORBT Money Market**](../pocket/what-is-a-money-market/) **(ORBTMM)**, allowing on-demand access to liquidity wherever it’s needed.

This **modularity** ensures that each functional component - minting, market-making, lending, or arbitrage - operates independently yet remains composable within the ORBT ecosystem.

### 2. Intent-Centric User Experience

Users express **what they want to achieve**, not **how** to achieve it.\
ORBT’s interface and protocol logic are built around [**intents**](../pocket/pocket-lifecycle-and-flows/intent-based-settlement-instant.md), abstracting away multi-step operations such as collateral management, bridging, and execution routing.

Behind each user action, ORBT automatically composes optimal transaction paths, leveraging on-chain liquidity and allocator strategies.\
This model mirrors the evolution from transaction-driven DeFi to **intent-driven finance (iFi)** - prioritizing simplicity and outcome over technical interaction.

### 3. Permissionless Core, Permissioned Edge

ORBT maintains a **dual trust model**:

* The **core protocol** is open-source, auditable, and permissionless for general users.
* The **edge layer**, involving [allocators](../allocator/), or institutional pockets, operates under governance approval and controlled access.

This allows ORBT to scale responsibly - combining DeFi’s openness with the compliance rigor needed for institutional-grade participation.

### 4. Transparency and Verifiability

Every major ORBT operation is **verifiable on-chain**:

* Collateralization ratios and positions are tracked publicly via subgraphs.
* All allocator interactions and Pocket executions are logged immutably.
* Treasury and buyback actions are performed by verifiable contracts, not opaque operators.

The system’s mantra: _“<mark style="background-color:green;">If it’s not on-chain, it’s on record.</mark>”_

This transparency builds user trust and meets institutional reporting requirements without sacrificing decentralization.

### 5. Composable Ecosystem Alignment

ORBT is designed to integrate seamlessly with other DeFi protocols and real-world infrastructures.\
By adhering to ERC standards, supporting cross-chain interoperability (via CCIP, LayerZero, etc.), and enabling data transparency through subgraphs and APIs, ORBT ensures that **its assets (**[**0xUSD**](../0xusd/)**,** [**0xAssets**](../0xassets.md)**)** and **its architecture** can plug into any ecosystem.

This composability allows developers and DAOs to extend ORBT’s framework, building on top of it rather than around it.
