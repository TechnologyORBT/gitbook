# Adding Assets

### Asset Onboarding Overview

Adding a new Asset(A) or saving asset (s0x, ERC-4626 wrapper around the instance’s 0x) is an **ADMIN-gated** operation. Before calling `addAsset(asset, family, pocket, reserveBps)`, run a risk/onboarding checklist:

* [**Family fit**](swaps.md#instrument-classes)**:** The asset’s risk, pricing unit, and oracle model must match the **UCE instance’s** `contractFamily (BTC/ETH/USD)`. Cross-family additions are **rejected**.
* **Custody (**[**pocket**](../../../protocol-mechanics/integration/pocket.md)**) readiness:** A production-ready **pocket** (ERC-4626 vault, vault proxy, or custody address) must exist and be funded/allowanced operationally. This pocket will receive the **non-reserve** share of inbound deposits for that asset.
* [**Reserve policy**](reserve-policy.md)**:** Choose a **per-asset `reserveBps`** (default 25% if unset elsewhere). Higher reserves improve instant redemption capacity; lower reserves increase yield but raise reliance on pocket pulls.
* [**Oracle plan**](backing-invariants.md#id-5-oracle-usage-pricing-scope-and-guards) **(for A→0x only):** Ensure Chainlink-style feeds (base or USD with base/USD fallback) are specified and healthy; define a **heartbeat** and an optional **mint haircut**. (Oracles are attached **after** asset addition via `setOracle`.)
* [**Fee posture**](swaps.md#id-6-fees-what-when-to-whom)**:** Decide whether to set **`tinBps`** (mint-side fee) for the new asset post-addition via `setAssetTinBps`.
* [**s0x assets**](../../../user-flow-and-guides/how-to-buy-s0xasset.md)**:** Even if the token is an ERC-4626 with `asset() == zeroX`, it must be **registered** so that swaps recognize it. Oracles are **not** needed for s0x assets; 0x↔S0x uses ERC-4626 accounting.

***

### What OrbtUCE Configures & Enforces (Guardrails)

When `addAsset(asset, family, pocket, reserveBps)` is called, OrbtUCE performs and persists the following:

* **Validation (hard reverts on failure):**
  * `asset != 0`, `pocket != 0`.
  * `family == contractFamily` (family mismatch is rejected).
  * Asset not already registered (duplicate additions are rejected).
  * `reserveBps ≤ 10_000` (100% cap).
* **State installation:**
  * Writes `assetCfg[asset] = { pocket, family, reserveBps }` and initializes pause=false.
  * **Decimals caching**: reads and caches `decimals()` for the token (0x is fixed at 18). This guarantees deterministic, gas-efficient normalization for previews, conversions, and redemption math.
  * Emits **`PocketSet(asset, old=0, new=pocket, amountTransferred=0)`** as the canonical onboarding event.
* **Separation of concerns (post-add ops):**
  * **Oracles:** set later via `setOracle(asset, baseFeed, usdFeed, heartbeat, mintHaircutBps, enabled)`. A→0x pricing will **revert** until a valid oracle configuration is in place (s0x assets do not use oracles).
  * **Fees & reserves:** can be tuned anytime per asset via `setAssetTinBps(asset, bps)` and `setAssetReserveBps(asset, bps)`.
  * **Pause controls:** per-asset **`pauseAsset`/`unpauseAsset`** provide an immediate kill-switch for that asset’s swap legs.
  * **Pocket rotation:** `setPocket(asset, newPocket)` migrates **up to** `min(balance, allowance)` from old to new custody; if allowance is low, migration is partial by design (no unsafe pulls).

**Operational implications:**

* Redemptions for the new asset are always **bounded by on-hand reserve + allowance-bounded pocket liquidity**; if insufficient, calls revert (no synthetic A is ever minted).
* A→0x becomes available only after oracles are enabled and non-stale; 0x→A remains **oracle-free** (decimals-only) once the asset is registered and funded.
* ADMIN currently owns the full lifecycle: onboarding, oracle attachment, fee/reserve tuning, pocket rotations, and pause management, each change emitting specific events to keep off-chain monitoring and audits deterministic.
