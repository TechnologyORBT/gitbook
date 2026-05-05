# Policy Governed Collateralization

ORBT’s **Policy-Governed Collateralization** framework ensures that every minted asset - including **0xUSD, 0xBTC, 0xETH**, and other 0xAssets - maintains a **1:1 hard peg** with its underlying collateral.\
This design guarantees that all synthetic assets are fully backed, instantly redeemable, and governed by transparent, on-chain policies that eliminate leverage and preserve system integrity.

### 1:1 Collateralized Minting

Users can mint 0xAssets by depositing equivalent-value collateral such as **stablecoins, BTC, ETH, SOL, or institutional-grade tokens** into ORBT vaults.\
Each 0xAsset minted represents a **direct, verifiable claim** on real collateral held within the protocol.

* **Minting Process:**
  1. The user deposits approved collateral into a vault governed by the **Unified Collateral Engine (**[**UCE**](../../orbt-design/uce/)**)**.
  2. The protocol validates the deposit and issues 0xAssets of equal value - maintaining a strict 1:1 parity.
  3. Users can [redeem](../../orbt-design/uce/concepts/swaps.md#id-0x-a-redemption-side) at any time, ensuring a constant peg between the minted asset and its underlying reserve.
* **No Overcollateralization, No Leverage:**\
  Every unit of 0xAsset exists only when an equivalent unit of underlying collateral is locked in the system.\
  This ensures predictable backing, transparent redemption, and eliminates risks from fractional or leveraged issuance.

### Transparent Policy Governance

Collateral issuance and redemption are governed by **ORBT’s on-chain policy engine**, which operates under DAO supervision.\
All collateral rules, minting limits, and redemption parameters are defined through **community-approved governance proposals** and executed automatically by protocol smart contracts.

* **Policy Parameters Include:**
  * Eligible collateral types and whitelist status
  * Per-asset minting caps and circulation limits
  * Collateral pool health thresholds and liquidity reserves
  * Bridge-specific custody limits (per chain)
* **Automated Enforcement:**\
  The UCE enforces these parameters programmatically - preventing unauthorized minting, unbacked issuance, or policy breaches.\
  Once a DAO proposal passes, its logic is encoded directly into the collateral management contracts through a [**timelocked**](../../orbt-design/allocator/governance-and-policy-layer-setup.md#id-5.3-timelock) **upgrade cycle**, ensuring both transparency and predictability.

### Segregated Collateral Pools

Each asset type is supported by **dedicated, non-rehypothecated vaults**, guaranteeing full traceability and eliminating cross-asset risk.

* **Asset Isolation:**\
  The collateral backing [0xUSD ](../../orbt-design/0xusd/)is distinct from that backing 0xBTC or 0xETH.\
  This ensures that liquidity events or redemption activity in one asset cannot affect others.
* **Vault Standardization:**\
  All vaults conform to the **ERC-4626 standard**, allowing composability across chains and seamless integration with ORBT’s Modular Stablecoin Framework (MSF).\
  Collateral stored in these vaults remains visible on-chain, and no asset can be reused or pledged elsewhere.

### Transparent Minting Caps and Limits

To preserve peg stability and prevent uncontrolled expansion, every 0xAsset is governed by explicit **minting caps** and **supply thresholds**, visible to all users.

* **Global Mint Caps:**\
  Define the total allowable circulation for each 0xAsset (e.g., total 0xUSD supply across all chains).
* **Collateral Utilization Limits:**\
  Each vault can only issue assets up to 100% of its available collateral - enforcing a hard stop against overextension.
* **Public Dashboards:**\
  Live subgraph dashboards display minted supply, collateral reserves, redemption activity, and chain-level distributions in real time.

This transparency ensures that all minted 0xAssets remain verifiably backed and fully redeemable on demand.

### Integration Across the 0xAsset Ecosystem

The 1:1 collateralization principle extends across all 0xAssets in the ORBT ecosystem:

* [**0xUSD**](../../orbt-design/0xusd/)**:** Fully collateralized stable asset, maintaining a dollar-equivalent peg via stablecoin and institutional reserves.
* **0xBTC / 0xETH:** Wrapped, fully collateral-backed representations of BTC and ETH, allowing composable liquidity across multiple chains.
* **Custom 0xAssets:** DAO-approved, asset-specific issuances following the same 1:1 collateralization and policy governance standards.

This unified model guarantees consistency, safety, and parity across all asset classes.

### Real-Time Proof and Auditability

The **Proof-of-Reserves system** continuously verifies that total collateral value equals or exceeds total issued supply: ensuring that the 1:1 peg remains intact across all chains.

* **Oracle Integration:**\
  Price and reserve data are sourced from **Chainlink oracles** and validated through **Oracle Security Modules (OSMs)** for time-delayed verification.
* **Public Proof Dashboards:**\
  On-chain subgraphs and analytics dashboards allow any user to verify total collateral, minted supply, and peg status in real time.
* **Independent Audits:**\
  Regular audits by third-party firms confirm parity between issued assets and underlying holdings, with full transparency reports published via the DAO.
