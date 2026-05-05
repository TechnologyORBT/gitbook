# ORBT

### ORBT - Unified Collateral Engine

#### Objective

Unify fragmented onchain liquidity across BTC, ETH, and USD families via a 1:1 standardized asset layer (0xAssets), programmable pockets, and allocator-driven credit lines, while preserving strict solvency and predictable settlement.

#### Components

* [Peg Stability Module](../uce/architecture.md#the-peg-and-liquidity-model) (PSM)
* [0xAssets](../0xassets.md) (0xUSD/0xETH/0xBTC)
* [Savings](../../protocol-mechanics/savings-rate.md)/Accrual wrapper (s0xAsset, ERC4626)
* [User Position Manager](../upm/) (UPM) for arbitrary safe execution
* [Pocket](../pocket/)
* [Allocator](../allocator/)
* [Strategy adapters](../pocket/aave-only-pocket-strategy/) (e.g., AaveSupplyOnly)
* [StakingRewards](../../protocol-mechanics/reward-rate.md#orbt-rewards-rate) for ORBT emissions
* [Governance](../allocator/governance-and-policy-layer-setup.md) with timelock and EIP-712 multisig.

#### **Core Properties**

Deterministic 1:1 accounting, allocator-scoped credit mint/repay, dynamic redemption fee with decay, referral-based inventory attribution, reserve policy on inbound flows, global vs allocator liquidity separation, and formalized pocket migration.
