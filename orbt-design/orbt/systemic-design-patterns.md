# Systemic Design Patterns

The systemic design of ORBT is built on **reusable, modular patterns** that allow efficient iteration, governance scalability, and interoperability.

These patterns define how different protocol components communicate, evolve, and safeguard against systemic risk.

### 1. The Pocket Pattern (Isolated Execution)

[Pockets ](../pocket/)are self-contained smart contracts that execute user or allocator strategies.\
Each Pocket operates with its own storage and access boundaries, ensuring that:

* No single strategy can drain global liquidity.
* Governance can limit exposure per Pocket (e.g., 2% of total assets).
* Bugs or losses are isolated to their instance.

This design mirrors the **“microservice” architecture** in software engineering, decentralizing execution to reduce systemic risk.

### 2. The Treasury Cycle Pattern

Revenue and surplus flow through a repeatable [**cycle**](../../protocol-mechanics/orbt-revenue-and-yield-model/orbt-flywheel-revenue-to-value-loop.md):

<mark style="background-color:yellow;">**1.Revenue accrual**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">→</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**2.Yield distribution**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">→</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**3.Buyback & burn**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">→</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**4. Savings rewards**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">→</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**5. Treasury recycling**</mark><mark style="background-color:yellow;">.</mark>

This ensures continuous value reinforcement (the [_**flywheel effect**_](../../protocol-mechanics/orbt-revenue-and-yield-model/orbt-flywheel-revenue-to-value-loop.md)) and eliminates manual intervention.\
The cycle is algorithmically managed by smart contracts, governed via periodic DAO votes to update distribution ratios.

### 3. The Permit-First UX Pattern

ORBT implements **EIP-2612 permits** and signature-based authorization across all key interactions.\
Users sign once to express intent; the system executes multiple steps behind the scenes - minting, swapping, deposits, etc.

This minimizes on-chain friction and aligns with the **intent-based design philosophy** of ORBT, making complex financial operations feel effortless.

### 4. The Dual-Layer Liquidity Pattern

ORBT’s liquidity is bifurcated into:

* **Active Liquidity** deployed across [Pockets ](../pocket/)and strategies to generate yield.
* **Reserve Liquidity** held in [UCE](../uce/) and [Safety Vaults](../../protocol-mechanics/orbt-revenue-and-yield-model/unified-capital-inflow.md#composable-vault-architecture) for instant redemptions and peg defense.

This ensures both **efficiency and solvency**: capital works continuously while maintaining user redemption confidence even during market stress.

### 5. The Governance-Limited Upgrade Pattern

Upgradability in ORBT is tightly controlled by governance with:

* Timelocked proposals,
* Version-controlled proxies, and
* Publicly auditable change history.

This allows ORBT to evolve safely, upgrading without trust erosion, while preventing unilateral control by any entity.

### 6. The Cross-Domain Synchronization Pattern

Because ORBT spans multiple chains, it synchronizes key states (like 0xUSD supply or allocator limits) via secure cross-chain messaging (CCIP, LayerZero).\
Each domain operates autonomously but contributes to a unified global state, much like shards in a distributed system.

This ensures **high fault tolerance**: even if one domain experiences issues, others continue operating safely with minimal systemic disruption.

### 7. The Transparency Feedback Pattern

Every transaction, yield event, and governance action emits standardized logs consumable by subgraphs and external APIs.\
Dashboards, analytics platforms, and institutional reporting tools can plug in seamlessly, ensuring that protocol data remains **observable, auditable, and provable**.

This transparency isn’t just documentation, it’s **part of ORBT’s systemic design**, keeping governance accountable and users informed.
