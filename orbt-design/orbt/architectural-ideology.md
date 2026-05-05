# Architectural Ideology

ORBT’s architecture blends **modularity**, **composability**, and **autonomous orchestration**.\
Its structure reflects lessons learned from leading DeFi systems like MakerDAO, Aave, and Angle, while introducing a **multi-layered liquidity and execution model** optimized for scale and flexibility.

### 1. Layered Architecture

The ORBT system is structured into **functional layers**, each with clear responsibility and isolation boundaries:

* **Base Layer (Assets & Collateral):** manages token standards, collateral vaults, and the [Unified Collateral Engine (UCE)](../uce/overview.md).
* **Liquidity Layer (**[**Money Market**](../pocket/what-is-a-money-market/) **& UCE):** coordinates lending, borrowing, and stablecoin swaps across chains.
* **Execution Layer (**[**Pockets**](../../protocol-mechanics/integration/pocket.md) **&** [**Allocators**](../uce/concepts/allocators/)**):** executes strategies, intents, and arbitrage opportunities.
* **Governance & Security Layer:** handles parameter updates, staking, buybacks, and risk controls.

This layered design ensures **fault isolation,** issues in one layer (e.g., strategy logic) cannot cascade into the others.\
It also enables parallel development and upgradability.

### 2. Cross-Chain Consciousness

Rather than relying on a single bridge or replicated deployments, ORBT employs a **cross-chain conscious architecture**.\
Every action, whether [minting 0xUSD](../../user-flow-and-guides/how-to-buy-0xassets.md) or [executing a Pocket](../pocket/pocket-lifecycle-and-flows/), is contextualized by chain identity and verified by ORBT’s coordination contracts.

This ensures:

* **Asset integrity** (no double issuance across chains),
* **Localized autonomy** (each chain instance can operate even if another is offline), and
* **Unified accounting** (governance tracks holistic system state across networks).

### 3. Unified Contract Design

All ORBT contracts follow a **template-driven pattern**.\
For example:

* The [**Pocket**](../pocket/) **Factory** standardizes execution strategies.
* The [**UPM**](../upm/) **(User Position Manager)** abstracts user vault logic.
* The [**UCE**](../uce/) **(Unified Collateral Engine)** maintains peg operations via consistent swap functions.

This shared interface approach ensures interoperability, allowing modules to be composed like building blocks while remaining individually auditable and upgradable.

### 4. Governance as a Design Primitive

Governance isn’t an afterthought, it’s **built into ORBT’s architecture**.\
Smart contracts expose roles, parameters, and hooks that can be adjusted through DAO votes or timelocked proposals.

This native governance integration allows:

* Dynamic [fee and rate adjustment](../uce/concepts/dynamic-redemption-fees.md),
* Allocator [onboarding/offboarding](../uce/concepts/allocators/#id-1.-onboarding-admin-governance),
* [Treasury](../system-overview.md#treasury-flows-and-economic-incentives) reallocation,
* [Emergency pause](../../resources/risk-disclaimer.md) and recovery operations.

In essence, governance defines **how the system evolves safely** without requiring redeployment or centralized interventions.

### 5. Deterministic Security

Security is embedded by design through:

* **Formal** [**audit**](../../resources/audit-and-security-disclosure.md) **workflows** and **verifiable contract invariants**,
* [**Permission segregation**](../system-overview.md#roles) (allocators, governance, users), and
* [**Automated fail-safes**](../../security-and-safety/security.md) (pause modules, rate limiters, Safety Vaults).

Each design choice reflects a principle of **deterministic containment,** meaning even under failure, the system’s impact is bounded and predictable.
