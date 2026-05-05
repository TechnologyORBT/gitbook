# Backing Invariants

Below is a consolidated list of **behavioral, accounting, liquidity, oracle, and control invariants** enforced by the `OrbtUCE` implementation you shared. These are phrased as properties that must always hold (or cause a revert) given the contract’s logic.

***

### 1) [Swap Plane & Pairing](swaps.md#instrument-classes)

* **Pair gating:** Only four pairs are ever valid: **A→0x, 0x→A, 0x→s0x, s0x→0x**. Any other pair (e.g., A↔A, 0x↔0x, s0x↔s0x, A→s0x, s0x→A) **reverts**.
* **Same-asset swap forbidden:** `assetIn == assetOut` **reverts**.
* **Allocator restriction:** Allocators **cannot** perform 0X→U swaps; attempts **revert**.

### 2) Asset Registration & Config

* **Asset validity:** An asset is “valid” if it’s the configured **0X** asset or it has a **non-zero pocket** registered in `assetCfg`. Otherwise, using it **reverts**.
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

* **Canonical precision:** 0x is **always 18 decimals**; conversions **only** use decimal scaling for 0x↔A math in the redemption leg and conversions.
* **Deterministic normalization:** `convertToAssets/convertToZeroXAssets` and all normalization helpers are pure-decimals transforms, **no oracle** used there.

### 5) Oracle Usage (Pricing Scope & Guards)

* **Oracle scope:** **Only** A→0x (mint side) uses oracles; 0x→A is **oracle-free** (decimals only).
* **Feed freshness:** For any used feed, price must be **positive**, **answeredInRound ≥ roundId**, and **not stale** relative to its `heartbeat`; otherwise **revert**.
* **USD/base composition:** If no base feed is present, asset USD feed and base/USD feed must both be present; otherwise **revert**.
* **Haircut application:** If `mintHaircutBps > 0`, 0x out on A→0x is reduced accordingly.

### 6) Fees: Correctness & Routing

* **Tin (mint-side fee):** Applied only on **A→0x**; reduces user’s **net 0x** and **mints fee in 0x to treasury**. If treasury is zero-address, fee mint is skipped (0x mint to treasury requires non-zero treasury).
* **Dynamic redemption fee:** Applied only on **0x→A**; calculated **in 0x** at a **snapshot rate** taken pre-settlement; **converted to underlying** and transferred to **treasury**.
* **Treasury presence for fee transfers:** If a **positive fee in underlying** must be paid and `treasury == 0`, function **reverts** (e.g., repay fee).
* **Fee limits:** Redemption fee rate is clipped to `MAX_REDEMPTION_RATE` (e.g., 5% cap). Floor ≥ 0.

### 7) Reserve & Pocket Liquidity

* **Per-asset reserve accrual:** On **A→0x**, `reservedUnderlying[asset]` increases by the configured reserve slice; the remainder is forwarded to the chosen pocket.
* **First source on redemption:** On **0x→A**, delivery **first consumes on-hand reserve** tracked in `reservedUnderlying[asset]` (not exceeding tracked units).
* **Bounded pocket pulls:** Any pull from a pocket (global or allocator) is limited by **min(balance, allowance)**; insufficiency **reverts**. No implicit borrowing or force-pulls.
* **No underlying mint:** Redemptions **never** mint underlying; all A sent must be sourced from on-hand or pocket pulls.

### 8) Settlement Parity & Execution Integrity

* **Preview parity for 0X→A:** The **snapshot** redemption rate is used for both preview and execution; delivered net A equals preview (modulo state changes between calls).
* **Exact-out enforcement:** For exact-out paths, delivered out amount **must equal** target; mismatch **reverts**.
* **ERC-4626 parity:** **0x↔s0x** uses `deposit/redeem/convertToShares/convertToAssets` consistently; if the shares/assets returned do not match expected (on exact-out), **revert**.
* **Settlement mismatch checks:** Internal assertions ensure the computed net and the transferred amounts align; otherwise **revert**.

### 9) Allocator Credit, Debt & Limits

* **Whitelisting & ACL:** Only **allocators or ADMIN** may credit-mint for a specific allocator; caller must be the allocator or ADMIN; otherwise **revert**.
* **Credit line existence:** Allocator credit-mint requires a non-zero **ceiling**; otherwise **revert**.
* **Daily cap (UTC-day):** `mintedToday` resets on new day; `mintedToday + amount ≤ dailyCap`. Violation **reverts**.
* **Ceiling (effective debt):** After credit-mint, **effective debt ≤ ceiling**; otherwise **revert**.
* **Repay asset type:** Repayment **must be in underlying** (not 0X). Using 0x **reverts**.
* **Repay capping:** Repayment applied **≤ current effective debt**. No over-repayment.
* **Borrow fee on repay:** If `borrowFeeBps > 0`, fee is taken **in underlying** to treasury before principal; if fee > 0 and treasury is zero, **revert**.
* **Epoch hygiene:** If allocator’s `debtEpoch` is stale (< `wipeEpoch`), it is synchronized before adding new debt; stale state does not leak.

### 10) Debt Indexing & Totals

* **Index positivity:** `debtIndex` is initialized to `1e18` and used as a positive scalar; all base↔effective conversions divide/multiply by `1e18`.
* **Effective debt formula:** For any allocator `a`, `effectiveDebt[a] = baseDebt[a] * debtIndex / 1e18`.
* **Total effective debt:** `totalAllocatorDebt() = baseTotalDebt * debtIndex / 1e18`.
* **Non-negativity:** `baseDebt[a]`, `baseTotalDebt`, `reservedZeroX`, `totalReservedZeroX`, and per-asset `reservedUnderlying[asset]` never underflow; all decreases are guarded/capped.
* **Index consistency:** All mutations that change allocator debt adjust **base** terms (`baseDebt`, `baseTotalDebt`), never directly editing effective terms.

### 11) Inventory & Pro-Rata Consumption

* **Unreserved first:** Protocol A→0x deliveries prefer **unreserved 0x on-hand** before tapping allocator inventories.
* **Pro-rata cap per allocator:** When pro-rata drawing allocator 0x for protocol A→0x, each allocator’s contribution is bounded by **`min(reservedZeroX, effectiveDebt)`**, ensuring inventory consumption never exceeds their debt capacity.
* **Debt netting:** Allocator 0x consumed by the protocol is treated as **repayment in 0x-equivalent**, reducing allocator `baseDebt` (via index math) and `baseTotalDebt`. Inventory usage **cannot increase** anyone’s debt.
* **Referral strictness:** For **referred A→0x**, 0x **must** come from the referral allocator’s `reservedZeroX`; insufficiency **reverts** (no socialization).

### 12) Redemption Fee Dynamics

* **Decay monotonicity:** Between events, the stored base rate decays multiplicatively per hour (bounded to a finite number of steps per read), never increasing during idle time.
* **Bump proportionality:** Post-trade bump is proportional to the **redeemed fraction of total 0x supply**; new rate is `min(cap, decayed + fraction)`.
* **Post-settlement update:** Rate decay/bump is applied **after** delivering funds and emitting the fee event, so the snapshot used for settlement remains immutable during that transaction.

### 13) Treasury & Emissions

* **Tin emission:** Mint-side 0x fee is **minted to treasury** (if set) and never deducted from reserves.
* **Redemption fee remittance:** Redemption fee is **transferred in underlying** to treasury.
* **No accidental drains:** Admin emergency withdraw and admin withdraws are bounded by **allowance & balance** of pockets; direct transfer of reserves to treasury requires explicit admin invocation and will **revert** if amounts are zero/invalid.

### 14) ERC-4626 Interactions

* **Asset binding for s0x assets:** An s0x asset is recognized only if `IERC4626(s).asset() == OX`. Otherwise, it’s not treated as an s0x asset.
* **Allowance hygiene:** Deposits use force-approve/reset patterns to avoid sticky allowances.

### 15) Access Control & Upgradability

* **ADMIN gating:** All configuration/admin functions (addAsset, setPocket, setTin, setOracle, setTreasury, pause/unpause, emergencyWithdraw, governance hooks, UUPS upgrade) require **ADMIN** role.
* **UUPS authorization:** `_authorizeUpgrade` enforces ADMIN-only upgrades.
* **Governance router:** Only the configured `governance` may invoke governance actions; unknown action types **no-op** (return false) without side effects.

### 16) Event Integrity

* **Swap/Deposit/Withdraw/Credit/Repaid:** Emitted with canonical fields after successful state transitions, allowing off-chain accounting to reconcile inputs/outputs and fee amounts.
* **Fee events:** `TinFeeTaken` and `RedemptionFeeTaken` log exact fee basis and units (gross, rate/bps, amounts, timestamps).

### 17) Derived (System-Level) Safety Properties

* **Backing visibility:** Redemptions are strictly bounded by **on-hand reserves + allowance-bounded pocket liquidity**; if insufficient, the call **reverts** (no hidden shortfall absorption).
* **Inventory solvency:** **Protocol** cannot “spend” allocator 0x beyond `totalReservedZeroX`; 0x deliveries use **unreserved**, then **allocator pro-rata**, then **mint**, while consistently updating `totalReservedZeroX`.
* **No price slippage on redemption:** 0x→A uses decimals only; user receives **exact unit-normalized underlying less the snapshot fee**.
* **Preview determinism:** With unchanged on-chain state between preview and swap, **preview ≈ execution** due to rate snapshotting and mirrored arithmetic.

***

**Note:** Some invariants are enforced as **hard reverts** (e.g., pairing, pause, bounds, caps, allowance), while others are **structural** (e.g., index scaling, pro-rata caps, source ordering). Together they form the contract’s solvency, fairness, and determinism guarantees.
