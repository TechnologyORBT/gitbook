# What is an Allocator?

Allocators are permissioned fund managers and distribution partners of the ORBT protocol. They:

* Bring liquidity (their own treasury, partner capital, or user funnels) into Orbt’s Unified Collateral Engine (UCE).
* Manage custody of underlying stablecoins via pockets (EOA, multisig, or strategy vaults) and keep allowances open so UCE can pull funds to settle user redemptions.
* Run conservative yield strategies on pocket balances (e.g., [Aave money markets](../pocket/aave-only-pocket-strategy/)) so idle inventory earns carry without compromising settlement readiness.
* Issue inventory: each allocator receives a [credit line](parts-to-deal-with.md#credit-line-and-inventory) to mint 0xAssets and warehouse that minted supply as reserved inventory (`reservedZeroX`) for settlement and distribution.

Allocators are the commercial surface of the protocol: they make primary markets in 0xAssets, monetize order flow, and convert yield + spread into business revenue while absorbing the operational risk of custody and liquidity provision.
