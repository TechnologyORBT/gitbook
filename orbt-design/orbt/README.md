# ORBT

### Orbit - Unified Collateral Engine

#### Objective

Unify fragmented onchain liquidity across BTC, ETH, and USD families via a 1:1 standardized asset layer (0xAssets), programmable pockets, and allocator-driven credit lines, while preserving strict solvency and predictable settlement.

#### Components

* Peg Stability Module (PSM)
* 0xAssets (OxUSD/OxETH/OxBTC)
* Staking/Accrual wrapper (sOxAsset, ERC4626)
* User Position Manager (UPM) for arbitrary safe execution
* Pocket
* Allocator
* Strategy adapters (e.g., AaveSupplyOnly)
* StakingRewards for ORBT emissions
* governance with timelock and EIP-712 multisig.

#### **Core Properties**

Deterministic 1:1 accounting, allocator-scoped credit mint/repay, dynamic redemption fee with decay, referral-based inventory attribution, reserve policy on inbound flows, global vs allocator liquidity separation, and formalized pocket migration.
