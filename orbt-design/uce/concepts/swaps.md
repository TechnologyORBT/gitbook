---
description: >-
  This section specifies the swap plane of the Unified Collateral Engine (UCE):
  what pairs are permitted, how quotes are computed, how liquidity is sourced
  and delivered, and when/where fees apply.
---

# Swaps

### 1) Overview

UCE enables programmatic conversion across three instrument classes:

* **U (Underlying):** external ERC-20s in the instance’s asset family (e.g., WBTC in the BTC family).
* **OX (0x Asset):** the family’s synthetic base (18 decimals).
* **S (s-asset):** ERC-4626 yield wrapper whose `asset()` is the family’s OX (e.g., `s0xBTC`).

**Swap plane (pair-gated):**

* **U → OX** (mint to OX)
* **OX → U** (redeem to underlying)
* **OX → S** (deposit OX into the ERC-4626 vault)
* **S → OX** (redeem s-shares back to OX)

**Not supported:** U ↔ U and OX ↔ OX. Unsupported pairs revert.

All user-facing swaps are non-reentrant, asset-pause aware, and validated for pair support and non-zero amounts.

***

***

### 2) Entry Points & Previews

#### Swap functions

* **`swapExactIn(assetIn, assetOut, amountIn, receiver, referralCode)`**\
  Consume an exact `amountIn`; deliver the computed `amountOut`.
* **`swapExactOut(assetIn, assetOut, amountOut, maxAmountIn, receiver, referralCode)`**\
  Target an exact `amountOut`; compute and cap the required `amountIn` against `maxAmountIn`.

Both routes emit a canonical `Swap` event with in/out amounts and the referral code.

#### Deterministic previews (no state change)

* **`previewSwapExactIn(assetIn, assetOut, amountIn) → amountOut`**
* **`previewSwapExactOut(assetIn, assetOut, amountOut) → amountIn`**

Previews replicate settlement math, including redemption-fee snapshots for OX→U and mint-side tin where applicable, so quote ≈ execution under identical state.

#### Conversions (utility)

* **`convertToAssets(asset, oxAmount)`**: 18-dec OX → asset decimals (pure decimals normalization).
* **`convertToOxAssets(asset, assets)`**: asset decimals → 18-dec OX (pure decimals normalization).

***

### 3) Pricing & Conversion Rules

#### U → OX (mint side)

* **Pricing source:** oracle priced in the UCE instance’s **asset family** (BTC/ETH/USD).
  * If a per-asset **base feed** exists → use it.
  * Else **USD feed ÷ base/USD feed** (both freshness-checked via heartbeats).
* **Decimals normalization:** all OX math at **18 decimals**; U assets are normalized into 18-dec OX space.
* **Mint haircut (optional):** per-asset `mintHaircutBps` reduces OX out defensively.
* **Tin (mint fee, optional):** per-asset `tinBps` reduces the user’s **net OX**; an equivalent **OX amount is minted to the treasury**.

**Exact-in formula (conceptual):**

```
stdIn      = normalizeTo18(assetIn, amountIn)
px         = priceInFamily1e18(assetIn)                 // oracle
oxPreTin   = stdIn * px / 1e18
oxAfterCut = oxPreTin * (1 - mintHaircutBps/1e4)
netOut     = oxAfterCut * (1 - tinBps/1e4)
feeOxTin   = oxAfterCut - netOut   // minted to treasury
```

**Exact-out:** computes the **gross pre-tin OX** required to deliver `amountOut` and inverts the pipeline (including haircut) to find `amountIn`.

#### OX → U (redemption side)

* **Pricing source:** **decimals normalization only** (no oracle).
* **Dynamic redemption fee:** charged **to the user** in OX-terms, then normalized to U for treasury delivery.
  * **Rate snapshot** is taken for the quote and used for execution to preserve preview parity.
  * After settlement the base rate **decays hourly** and is **bumped** by the redeemed fraction of OX supply, capped at **5%**.

**Exact-in formula (conceptual):**

```
feeRate    = redemptionRateSnapshot()         // 0 … 5% (1e18 scale)
feeOx      = oxIn * feeRate
grossU     = normalizeFrom18(assetOut, oxIn)
feeU       = normalizeFrom18(assetOut, feeOx)
netU       = grossU - feeU
```

**Exact-out:** compute `grossOx = underlyingToOx(assetOut, amountOut)` and add `feeOx = grossOx * feeRateSnapshot`.

#### OX ↔ S (ERC-4626)

* **OX → S:** `IERC4626(s).deposit(oxIn, receiver)` ⇒ shares minted per vault exchange rate.
* **S → OX:** `IERC4626(s).redeem(sharesIn, address(this), address(this))` ⇒ OX assets released.
* No UCE fee on these legs. Pricing is the ERC-4626 vault’s exchange rate.

***

### 4) Liquidity Sourcing & Delivery

#### Inbound U custody (U → OX)

When U is received, UCE:

1. **Keeps a reserve slice** on-contract: `reserveBps` (default 25%) — fast liquidity for redemptions.
2. **Forwards the remainder to a pocket** chosen by routing (see Referral & Routing).
   * Pockets are generic custody addresses; later **pulls are bounded by pocket balance and allowance**.

#### Outbound OX delivery (U → OX)

* **Protocol path** (no referral and caller not an allocator):
  1. Send **unreserved OX on-hand**.
  2. **Pro-rata draw** from allocators’ `reservedOx` **capped by each allocator’s effective debt** (netting reduces their debt).
  3. **Mint any shortfall** to the receiver.
* **Allocator/referral path:** OX is drawn from the **allocator’s `reservedOx`**; insufficient balance **reverts** (no silent socialization).

#### Outbound U delivery (OX → U)

* Deliver U to the receiver by pulling in order:
  1. **On-hand reserves** (consuming tracked reserved units first),
  2. **Referral allocator pocket** (if referral present),
  3. **Global pocket**.
* Pulls are strictly **bounded by allowance & balance**; if insufficient, the swap **reverts** (there is no “mint underlying” path).

***

### 5) Referral & Routing

* A **referral code** maps to a specific allocator.
* **U → OX with referral:**
  * **Underlying** is forwarded to the **allocator’s pocket** (if configured; otherwise global pocket).
  * **OX** is delivered **from the allocator’s `reservedOx`** to the user.
  * If the allocator’s `reservedOx` < required amount, the swap **reverts** (enforces flow ownership).
* **OX → U with referral:** U delivery preferentially pulls from the **referral allocator’s pocket**; if insufficient, falls back to global context (still bounded by allowance/balance).

Allocators **cannot** perform OX → U themselves; they must operate via underlying-side flows.

***

### 6) Exact-In vs Exact-Out Semantics

* **Exact-in** guarantees consumed input; the engine computes and enforces the precise output:
  * **U → OX:** applies oracle pricing, mint haircut, and tin; then settles OX outbound as per routing rules.
  * **OX → U:** snapshots redemption fee; delivers U using deterministic sourcing; routes the fee to treasury in U.
  * **OX ↔ S:** uses ERC-4626 `convertToShares/convertToAssets` paths to compute output.
* **Exact-out** guarantees delivered output; the engine computes the minimum input required:
  * **Caps input** with `maxAmountIn`; exceeding the cap reverts.
  * **Settlement parity checks** ensure the delivered amount matches the target; mismatches revert.

***

### 7) Fees: What, When, To Whom

* **Tin (mint fee)** — **U → OX** only; reduces user’s OX out and mints the fee in OX to **treasury**.\
  Event: `TinFeeTaken(payer, assetIn, oxGrossBeforeTin, tinBps, feeOx, timestamp)`.
* **Dynamic redemption fee** — **OX → U** only; charged to user at a **snapshot rate**, delivered to **treasury in underlying**.\
  Event: `RedemptionFeeTaken(payer, assetOut, oxIn, feeRate, feeInUnderlying, timestamp)`.
* **No UCE fee** on **OX ↔ S**.

***

### 8) Pause, Safety, and Guard Rails

* **Global pause** and **per-asset pause** halt relevant swap legs gracefully.
* **Non-reentrancy** on state-changing flows.
* **Oracle guardrails** (heartbeat freshness, non-negative, non-stale) on U → OX.
* **Pocket pulls bounded** by allowance & balance; insufficient pocket liquidity **reverts** rather than over-pulling.
* **Allocator restrictions:** OX → U is disallowed for allocators; daily caps and ceilings constrain credit behavior on inventory creation (separate from swaps but relevant to settlement sources).

***

### 9) Failure & Edge-Case Handling

* **Unsupported pairs** → revert.
* **Zero amounts** or **zero receiver** → revert.
* **Asset paused** → revert.
* **Insufficient pocket allowance/balance** during U delivery → revert.
* **Allocator/referral OX shortfall** in U → OX → revert (flow ownership).
* **Exact-out exceeding `maxAmountIn`** → revert.
* **Settlement mismatch** (e.g., ERC-4626 returns unexpected shares/assets) → revert.

***

### 10) Practical Implications

* **Peg discipline:** Redemptions are oracle-free and capacity-bounded; a dynamic fee smooths surges and self-decays.
* **Capital efficiency:** Protocol uses unreserved OX first; then nets **pro-rata** against allocators’ inventory (reducing their debt) before minting shortfall — minimizing new issuance.
* **Clear incentives:** Referral routes couple user inflows to allocator pockets and inventory; shortages are not socialized.
* **Deterministic quoting:** Previews match execution via rate snapshots and mirrored tin/decimals logic.

***

#### Quick Reference

* **Allowed pairs:** U→OX, OX→U, OX→S, S→OX.
* **U→OX:** Oracle-priced, optional haircut, optional tin to treasury; reserves kept on-hand; remainder to pocket; OX outbound = unreserved → pro-rata allocator inventory → mint.
* **OX→U:** Decimals normalization; dynamic fee (snapshot) to treasury; U sourced = reserves → referral pocket → global pocket; no underlying mint.
* **OX↔S:** ERC-4626 deposit/redeem; no UCE fee.
* **Referrals:** Attribute U inflows to allocator pockets and consume allocator `reservedOx`; shortages revert.
* **Safety:** Pair gating, pauses, heartbeats, allowance-bounded pulls, non-reentrancy
