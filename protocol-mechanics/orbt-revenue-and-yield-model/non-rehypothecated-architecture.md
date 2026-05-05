# Non Rehypothecated Architecture

ORBT’s architecture is founded on the principle of **true asset ownership,** ensuring that every deposit remains fully intact, traceable, and immune to rehypothecation or hidden leverage.\
All collateral and liquidity positions are managed through **isolated, auditable receipt structures**, giving users verifiable proof of custody at all times.

### Isolated Receipt-Based Custody

Every asset deposited into ORBT, whether stablecoins, BTC, ETH, SOL, or institutional tokens, is held in **segregated vaults** that issue an on-chain receipt token representing direct ownership of the underlying deposit.

* **One-to-One Receipts:**\
  Each deposit generates a corresponding ERC-4626-compliant receipt token that reflects a 1:1 claim on the specific asset it represents.
* **No Pledging or Pool Reuse:**\
  Assets are never lent, pledged, or rehypothecated for external borrowing or liquidity purposes. The only time an asset moves is when the original owner redeems it or executes a permitted transaction under protocol rules.
* **Vault-Level Isolation:**\
  Every vault operates independently, ensuring that operational or market events affecting one asset cannot cascade into others.

This model preserves **direct user control** and eliminates systemic risk from hidden leverage or unauthorized reuse of collateral.

### Transparent and Auditable Structures

ORBT’s **receipt architecture** is fully verifiable on-chain.\
Each vault publishes real-time data on asset balances, receipt supply, and redemption activity, creating an immutable audit trail accessible to anyone.

* **On-Chain Proofs:**\
  Subgraphs and Proof-of-Reserve dashboards show exact backing for every issued receipt.
* **Independent Verification:**\
  External auditors and the DAO’s Risk Committee can verify parity between vault holdings and outstanding receipts at any time.
* **Event Logging:**\
  All inflows, withdrawals, and policy updates are logged immutably, ensuring that asset flows can be traced without ambiguity or off-chain reconciliation.

Through this transparency, ORBT establishes an auditable, non-custodial environment suitable for both retail and institutional participants.

### Immutable Ownership Logic

Ownership within ORBT is **immutable by design,** governed entirely by smart contracts rather than intermediaries.

* **Code-Enforced Rights:**\
  Users retain withdrawal rights directly in contract logic. No admin keys or  allocator permissions can override, pause, or redirect funds.
* **Non-Abstracted Balances:**\
  The system never issues derivative or abstracted IOUs that represent pooled assets. Every receipt maps directly to a real deposit, ensuring clarity between user balance and actual holdings.
* **DAO-Controlled Policy Layer:**\
  Governance defines operational rules, such as eligible assets or withdrawal conditions, but cannot seize or repurpose user deposits.

This immutable custody design guarantees **full user sovereignty** while maintaining institutional-grade auditability.

### Integration with the [Unified Collateral Engine](../../orbt-design/uce/) (UCE)

The non-rehypothecated structure integrates seamlessly into ORBT’s **Unified Collateral Engine**.\
Each vault feeds verified liquidity into the UCE without relinquishing ownership integrity, enabling composability while preserving isolation.

* Liquidity used for minting 0xAssets or funding allocators remains **policy-bound and traceable**.
* Capital cannot be “double-counted” across modules; every asset’s utilization is logged and capped at its actual deposit value.
* Any yield or settlement activity through the Modular Stablecoin Framework (MSF) is executed using **explicit, permissioned access layers** that reference these underlying receipts.

Thus, ORBT achieves **capital efficiency without compromising custody discipline**.
