# Intent Routed Execution

ORBT’s **Intent-Routed Execution** framework redefines how transactions are processed across networks, prioritizing efficiency, transparency, and best execution outcomes.

By integrating directly with **solver networks** such as **UniswapX**, **CoW Protocol**, and other intent-based routing systems, ORBT enables **cross-chain, low-slippage settlement** that aligns user intent with optimal liquidity paths.

### Intent-Based Transaction Model

Traditional transaction execution relies on users defining exact trade routes and parameters. ORBT replaces this with an [**intent-based model**](../../orbt-design/pocket/pocket-lifecycle-and-flows/intent-based-settlement-instant.md), where users express _what outcome they want_, and the protocol’s integrated solver network determines _how to achieve it_.

* **User Intent:**\
  A user specifies their target - e.g., swap 0xUSD to USDC, settling a cross-chain payment, or rebalancing collateral.
* **Solver Discovery:**\
  ORBT broadcasts this intent to a network of solvers (UniswapX, CoW, or permissioned allocators) who compete to fulfill it at the best price, lowest gas cost, and minimal slippage.
* **Execution Assurance:**\
  The winning solver executes the transaction under the protocol’s settlement rules, guaranteeing the user receives their intended outcome without manual routing or bridging steps.

This design creates a **frictionless execution layer** that blends efficiency from DeFi’s open liquidity networks with ORBT’s institutional-grade guarantees.

### Integration with Solver Networks

ORBT’s integration with established solver infrastructures expands its reach across multiple liquidity venues and chains:

* **UniswapX:** Enables decentralized auction-based routing that aggregates liquidity from AMMs, limit orders, and cross-chain pools.
* **CoW Protocol:** Offers batch auctions that match opposing user intents, reducing MEV and ensuring minimal slippage.
* **Permissioned Solvers:** Institutional solvers approved by ORBT governance can participate in settlement routing for high-volume or regulated transactions, ensuring compliance and transparency.

Each solver operates within **policy-defined parameters**, ensuring execution integrity, cost efficiency, and consistency with ORBT’s governance standards.

### Institutional-Grade Settlement Guarantees

Intent-based routing within ORBT is designed for **institutional predictability** and **regulatory transparency**.

* **Deterministic Pricing:** Execution occurs under fixed settlement logic, preventing hidden spreads or discretionary intervention.
* **Pre-Funded Execution:** All transactions draw from pre-funded liquidity within the UCE, eliminating counterparty risk and settlement failure.
* **Auditable Traceability:** Each execution path is recorded on-chain, providing full traceability for compliance and post-trade reporting.

This ensures that even complex transactions retain **finality, auditability, and compliance compatibility**.
