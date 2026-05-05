# Backing Invariants

Below is a consolidated list of **behavioral, accounting, liquidity, oracle, and control invariants** enforced by the `OrbtUCE` implementation you shared. These are phrased as properties that must always hold (or cause a revert) given the contract’s logic.

***

### 1) Swap Plane & Pairing

* **Pair gating:** Only four pairs are ever valid: **U→OX**, **OX→U**, **OX→S**, **S→OX**. Any other pair (e.g., U↔U, OX↔OX, S↔S, U→S, S→U) **reverts**.
* **Same-asset swap forbidden:** `assetIn == assetOut` **reverts**.
* **Allocator restriction:** Allocators **cannot** perform OX→U swaps; attempts **revert**.

### 2) Asset Registration & Config

* **Asset validity:** An asset is “valid” iff it’s the configured **OX** asset or it has a **non-zero pocket** registered in `assetCfg`. Otherwise, using it **reverts**.
* **Family consistency:** New assets can be added only if `family == contractFamily`; otherwise **revert**.
* **Reserve ratio bounds:** `reserveBps ≤ 10_000` (100%). Attempts to exceed **revert**.
* **Tin bounds:** `tinBps ≤ 10_000`. Attempts to exceed **revert**.
* **Pocket updates:** Changing pockets requires non-zero new pocket and distinct from old; same pocket **reverts**. Migration transfers **up to allowance & balance** only.

### 3) Pause & Safety

* **Global & per-asset pause:** Any state-changing flow involving a paused asset **reverts**; global pause halts guarded functions.
* **Non-reentrancy:** Swap and repay/withdraw paths enforce `nonReentrant`.
* **Receiver sanity:** `receiver != address(0)` for swaps; otherwise **revert**.
* **Amount sanity:** Zero amounts for relevant functions **revert** (e.g., swap amounts, repay amounts).

### 4) Decimals & Normalization

* **Canonical precision:** OX is **always 18 decimals**; conversions **only** use decimal scaling for OX↔U math in the redemption leg and conversions.
* **Deterministic normalization:** `convertToAssets/convertToOxAssets` and all normalization helpers are pure-decimals transforms—**no oracle** used there.

### 5) Oracle Usage (Pricing Scope & Guards)

* **Oracle scope:** **Only** U→OX (mint side) uses oracles; OX→U is **oracle-free** (decimals only).
* **Feed freshness:** For any used feed, price must be **positive**, **answeredInRound ≥ roundId**, and **not stale** relative to its `heartbeat`; otherwise **revert**.
* **USD/base composition:** If no base feed is present, asset USD feed and base/USD feed must both be present; otherwise **revert**.
* **Haircut application:** If `mintHaircutBps > 0`, OX out on U→OX is reduced accordingly.

### 6) Fees — Correctness & Routing

* **Tin (mint-side fee):** Applied only on **U→OX**; reduces user’s **net OX** and **mints fee in OX to treasury**. If treasury is zero-address, fee mint is skipped (OX mint to treasury requires non-zero treasury).
* **Dynamic redemption fee:** Applied only on **OX→U**; calculated **in OX** at a **snapshot rate** taken pre-settlement; **converted to underlying** and transferred to **treasury**.
* **Treasury presence for fee transfers:** If a **positive fee in underlying** must be paid and `treasury == 0`, function **reverts** (e.g., repay fee).
* **Fee limits:** Redemption fee rate is clipped to `MAX_REDEMPTION_RATE` (e.g., 5% cap). Floor ≥ 0.

### 7) Reserve & Pocket Liquidity

* **Per-asset reserve accrual:** On **U→OX**, `reservedUnderlying[asset]` increases by the configured reserve slice; the remainder is forwarded to the chosen pocket.
* **First source on redemption:** On **OX→U**, delivery **first consumes on-hand reserve** tracked in `reservedUnderlying[asset]` (not exceeding tracked units).
* **Bounded pocket pulls:** Any pull from a pocket (global or allocator) is limited by **min(balance, allowance)**; insufficiency **reverts**. No implicit borrowing or force-pulls.
* **No underlying mint:** Redemptions **never** mint underlying; all U sent must be sourced from on-hand or pocket pulls.

### 8) Settlement Parity & Execution Integrity

* **Preview parity for OX→U:** The **snapshot** redemption rate is used for both preview and execution; delivered net U equals preview (modulo state changes between calls).
* **Exact-out enforcement:** For exact-out paths, delivered out amount **must equal** target; mismatch **reverts**.
* **ERC-4626 parity:** OX↔S uses `deposit/redeem/convertToShares/convertToAssets` consistently; if the shares/assets returned do not match expected (on exact-out), **revert**.
* **Settlement mismatch checks:** Internal assertions ensure the computed net and the transferred amounts align; otherwise **revert**.

### 9) Allocator Credit, Debt & Limits

* **Whitelisting & ACL:** Only **allocators or ADMIN** may credit-mint for a specific allocator; caller must be the allocator or ADMIN; otherwise **revert**.
* **Credit line existence:** Allocator credit-mint requires a non-zero **ceiling**; otherwise **revert**.
* **Daily cap (UTC-day):** `mintedToday` resets on new day; `mintedToday + amount ≤ dailyCap`. Violation **reverts**.
* **Ceiling (effective debt):** After credit-mint, **effective debt ≤ ceiling**; otherwise **revert**.
* **Repay asset type:** Repayment **must be in underlying** (not OX). Using OX **reverts**.
* **Repay capping:** Repayment applied **≤ current effective debt**. No over-repayment.
* **Borrow fee on repay:** If `borrowFeeBps > 0`, fee is taken **in underlying** to treasury before principal; if fee > 0 and treasury is zero, **revert**.
* **Epoch hygiene:** If allocator’s `debtEpoch` is stale (< `wipeEpoch`), it is synchronized before adding new debt; stale state does not leak.

### 10) Debt Indexing & Totals

* **Index positivity:** `debtIndex` is initialized to `1e18` and used as a positive scalar; all base↔effective conversions divide/multiply by `1e18`.
* **Effective debt formula:** For any allocator `a`, `effectiveDebt[a] = baseDebt[a] * debtIndex / 1e18`.
* **Total effective debt:** `totalAllocatorDebt() = baseTotalDebt * debtIndex / 1e18`.
* **Non-negativity:** `baseDebt[a]`, `baseTotalDebt`, `reservedOx`, `totalReservedOx`, and per-asset `reservedUnderlying[asset]` never underflow; all decreases are guarded/capped.
* **Index consistency:** All mutations that change allocator debt adjust **base** terms (`baseDebt`, `baseTotalDebt`), never directly editing effective terms.

### 11) Inventory & Pro-Rata Consumption

* **Unreserved first:** Protocol U→OX deliveries prefer **unreserved OX on-hand** before tapping allocator inventories.
* **Pro-rata cap per allocator:** When pro-rata drawing allocator OX for protocol U→OX, each allocator’s contribution is bounded by **`min(reservedOx, effectiveDebt)`**—ensuring inventory consumption never exceeds their debt capacity.
* **Debt netting:** Allocator OX consumed by the protocol is treated as **repayment in OX-equivalent**, reducing allocator `baseDebt` (via index math) and `baseTotalDebt`. Inventory usage **cannot increase** anyone’s debt.
* **Referral strictness:** For **referred U→OX**, OX **must** come from the referral allocator’s `reservedOx`; insufficiency **reverts** (no socialization).

### 12) Redemption Fee Dynamics

* **Decay monotonicity:** Between events, the stored base rate decays multiplicatively per hour (bounded to a finite number of steps per read), never increasing during idle time.
* **Bump proportionality:** Post-trade bump is proportional to the **redeemed fraction of total OX supply**; new rate is `min(cap, decayed + fraction)`.
* **Post-settlement update:** Rate decay/bump is applied **after** delivering funds and emitting the fee event—so the snapshot used for settlement remains immutable during that transaction.

### 13) Treasury & Emissions

* **Tin emission:** Mint-side OX fee is **minted to treasury** (if set) and never deducted from reserves.
* **Redemption fee remittance:** Redemption fee is **transferred in underlying** to treasury.
* **No accidental drains:** Admin emergency withdraw and admin withdraws are bounded by **allowance & balance** of pockets; direct transfer of reserves to treasury requires explicit admin invocation and will **revert** if amounts are zero/invalid.

### 14) ERC-4626 Interactions

* **Asset binding for S-assets:** An S-asset is recognized only if `IERC4626(s).asset() == OX`. Otherwise, it’s not treated as an S-asset.
* **Allowance hygiene:** Deposits use force-approve/reset patterns to avoid sticky allowances.

### 15) Access Control & Upgradability

* **ADMIN gating:** All configuration/admin functions (addAsset, setPocket, setTin, setOracle, setTreasury, pause/unpause, emergencyWithdraw, governance hooks, UUPS upgrade) require **ADMIN** role.
* **UUPS authorization:** `_authorizeUpgrade` enforces ADMIN-only upgrades.
* **Governance router:** Only the configured `governance` may invoke governance actions; unknown action types **no-op** (return false) without side effects.

### 16) Event Integrity

* **Swap/Deposit/Withdraw/Credit/Repaid:** Emitted with canonical fields after successful state transitions—allowing off-chain accounting to reconcile inputs/outputs and fee amounts.
* **Fee events:** `TinFeeTaken` and `RedemptionFeeTaken` log exact fee basis and units (gross, rate/bps, amounts, timestamps).

### 17) Derived (System-Level) Safety Properties

* **Backing visibility:** Redemptions are strictly bounded by **on-hand reserves + allowance-bounded pocket liquidity**; if insufficient, the call **reverts** (no hidden shortfall absorption).
* **Inventory solvency:** **Protocol** cannot “spend” allocator OX beyond `totalReservedOx`; OX deliveries use **unreserved**, then **allocator pro-rata**, then **mint**, while consistently updating `totalReservedOx`.
* **No price slippage on redemption:** OX→U uses decimals only; user receives **exact unit-normalized underlying less the snapshot fee**.
* **Preview determinism:** With unchanged on-chain state between preview and swap, **preview ≈ execution** due to rate snapshotting and mirrored arithmetic.

***

**Note:** Some invariants are enforced as **hard reverts** (e.g., pairing, pause, bounds, caps, allowance), while others are **structural** (e.g., index scaling, pro-rata caps, source ordering). Together they form the contract’s solvency, fairness, and determinism guarantees.
