# Immutable Buyback Engine

The **Immutable Buyback Engine** serves as ORBT’s deflationary and value-alignment mechanism -  automatically linking protocol performance to token value.

Whenever the protocol generates **net positive yield**, the system triggers an on-chain [**buyback-and-burn**](../../introduction/orbt/#id-2-value-capture-buyback-and-burn) **cycle**, permanently reducing token supply and reinforcing the ecosystem’s growth flywheel.

### Automated Yield Conversion

All revenue sources within ORBT - including settlement fees, allocator profit shares, and yield from the Unified Collateral Engine (UCE) - are continuously tracked through the protocol’s accounting module.

* **Net Yield Detection:**\
  The buyback engine activates only when net protocol income exceeds predefined operational thresholds.
* **Automated Conversion:**\
  Surplus yield, denominated in stablecoins or other base assets, is automatically converted into [**ORBT tokens**](../../introduction/orbt/) via decentralized exchanges or integrated liquidity pools.
* **Execution Integrity:**\
  The buyback process is governed entirely by immutable smart contracts, ensuring that no entity or governance proposal can redirect funds or pause execution once triggered.

This automation ensures **consistent, transparent, and policy-driven buyback activity**, tightly coupled to actual protocol performance.

### Burn and Supply Reduction

Once ORBT tokens are repurchased, they are **irreversibly burned** - removing them permanently from circulation.

* **On-Chain Burn Verification:**\
  Every burn event is verifiable on-chain through public transaction records.
* **Deflationary Dynamics:**\
  Reduced circulating supply strengthens long-term token value and aligns the interests of token holders, allocators, and stakers.
* **Governance-Protected Logic:**\
  The burn mechanism is encoded as immutable contract logic, ensuring that neither the DAO nor developers can divert or reallocate buyback reserves.

This guarantees that every cycle of ecosystem success translates directly into **token scarcity and value reinforcement**.

### Ecosystem Flywheel

The Immutable Buyback Engine fuels ORBT’s [**self-reinforcing growth loop**](orbt-flywheel-revenue-to-value-loop.md), where economic activity continuously drives ecosystem expansion:

1. **Protocol Growth:** Transaction volume, liquidity utilization, and yield generation increase.
2. **Yield Accumulation:** Net profits flow into the buyback module.
3. **Token Buyback & Burn:** The system automatically reduces supply, strengthening token value.
4. **Stakeholder Incentives:** Higher token value and yield performance attract more users, liquidity, and institutional participants.

This cycle forms a **sustainable feedback loop**, ensuring that ORBT’s growth translates into measurable value for participants while maintaining financial discipline through immutable automation.

### Transparency and Reporting

Every buyback-and-burn event is **publicly logged** and reflected in ORBT’s performance dashboards:

* **On-Chain Records:** Show total yield allocated to buybacks, execution timestamps, and burned token quantities.
* **Treasury Statements:** The DAO treasury publishes periodic summaries showing buyback frequency, total expenditure, and cumulative burn data.
* **Immutable Proof:** Since the buyback logic is hard-coded and non-upgradeable, the protocol’s deflationary operations remain permanently verifiable.

This creates a **fully auditable, policy-driven deflation model**, fostering trust and accountability at both community and institutional levels.
