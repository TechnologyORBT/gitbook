# Capital Protection

Capital protection is the foundation of ORBT’s security and liquidity model. Every design choice, from collateral management to reserve allocation, is aimed at preserving user funds while maintaining predictable liquidity across markets.

### Secured Liquidity Layer

All assets within ORBT are aggregated into a [**Unified Collateral Engine**](../orbt-design/uce/)**(UCE)** - a policy-controlled, on-chain liquidity pool that serves as the protocol’s financial core.\
The UCE ensures:

* **Full transparency:** All balances and liabilities are verifiable on-chain.
* **Automated enforcement:** Collateral rules, withdrawal limits, and rebalancing are enforced by code, not by intermediaries.
* **Deterministic control:** Governance policies determine allocation ratios, reserve buffers, and yield strategies without exposing user assets to discretionary risk.

This unified structure enables capital efficiency while maintaining strict separation between user-owned collateral and protocol-managed liquidity.

### Segregated Custody Architecture

User assets are stored in **non-rehypothecated vaults** based on the ERC-4626 tokenized vault standard.\
Each vault represents a self-contained, auditable unit of ownership, guaranteeing that:

* Deposited assets are never lent, pledged, or used as collateral elsewhere.
* Withdrawals follow deterministic accounting rules.
* Allocators and governance entities cannot reassign or pool collateral outside protocol policy.

This segregation ensures transparent asset tracking, simplifies audits, and eliminates hidden leverage or reuse risk common in opaque custodial systems.

### Reserve Buffer and Instant Settlement

To maintain immediate redemption capability even during volatile market conditions, ORBT maintains a [**reserve buffer**](../orbt-design/pocket/reserve-buffer-and-rebalancing.md) in liquid assets.\
This buffer guarantees:

* **Instant settlement** for withdrawals and redemptions.
* **Reduced slippage** during high-volume transactions.
* **Operational continuity** during stress events, bridge delays, or oracle disruptions.

The reserve buffer acts as a dynamic safeguard - automatically replenished through protocol inflows and rebalanced through the Unified Collateral Engine.

### Yield Optimization via MSF

Liquidity exceeding the reserve threshold is intelligently deployed into **low-risk, yield-generating strategies** via the [**Modular Stablecoin Framework (MSF)**](../protocol-mechanics/orbt-revenue-and-yield-model/policy-governed-collateralization.md#segregated-collateral-pools).\
These strategies focus on:

* **Short-duration DeFi money markets**, pre-approved through governance.
* **Capital preservation first**, with real-time monitoring of position exposure.
* **Automated risk throttling**, ensuring that capital can be recalled instantly in case of market distress.

This dual-layer approach - reserving core liquidity while optimizing excess capital - allows ORBT to maintain solvency and profitability without compromising user protection.
