---
description: >-
  This section specifies the swap plane of the Unified Collateral Engine (UCE):
  what pairs are permitted, how quotes are computed, how liquidity is sourced
  and delivered, and when/where fees apply.
---

# Swaps

## Instrument Classes

**Unified Collateral Engine (UCE)** enables seamless, programmatic conversion between three standardized instrument classes within an asset family. Each class maintains a clear functional role in the system, ensuring deterministic and safe conversion paths.

<table><thead><tr><th width="128.11114501953125">Symbol</th><th width="163.22222900390625">Name</th><th>Description</th></tr></thead><tbody><tr><td><strong>A (Asset)</strong></td><td><strong>Underlying Asset</strong></td><td>Represents the external ERC-20 token within a specific asset family. These are native or wrapped blue-chip assets such as <strong>BTC</strong>, <strong>ETH</strong>, or <strong>wETH</strong>. Assets originate outside the protocol. <em>Example:</em> <code>A = WBTC</code></td></tr><tr><td><strong>0x (Protocol Asset)</strong></td><td><strong>Protocol Synthetic Base</strong></td><td>Represents the <strong>protocol’s synthetic version</strong> of the asset. It is an <strong>18-decimal ERC-20 token</strong> that maintains a <strong>1:1 hard peg</strong> with its corresponding Asset (<strong>A</strong>). <em>Example:</em> <code>0xBTC</code> ↔ <code>WBTC</code></td></tr><tr><td><strong>s0x (Saving 0x Asset)</strong></td><td><strong>ERC-4626 Yield Wrapper</strong></td><td>Represents the <strong>yield-bearing receipt token</strong> obtained when 0x is deposited into the protocol’s <strong>savings module</strong> (ERC-4626 vault). The vault’s <code>asset()</code> corresponds to the 0x token. <em>Example:</em> <code>s0xBTC</code> represents deposited <code>0xBTC</code>.  </td></tr></tbody></table>

#### **Swap Planes (Pair-Gated Conversion)**

All conversions are validated for supported pairs and non-zero amounts. Unsupported pairs **revert** automatically.\
User-facing swaps are **non-reentrant**, **asset-pause aware**, and strictly validated for **pair support**.

<table><thead><tr><th width="132.5555419921875">Direction</th><th width="124.77789306640625">Action</th><th>Description</th></tr></thead><tbody><tr><td><strong>A → 0x</strong></td><td><strong>Mint</strong></td><td>Converts an external underlying asset (A) into its protocol representation (0x).</td></tr><tr><td><strong>0x → A</strong></td><td><strong>Redeem</strong></td><td>Redeems a 0x asset back to its original underlying asset (A).</td></tr><tr><td><strong>0x → s0x</strong></td><td><strong>Deposit</strong></td><td>Deposits 0x into the ERC-4626 savings vault to mint s0x (receipt tokens).</td></tr><tr><td><strong>s0x → 0x</strong></td><td><strong>Withdraw</strong></td><td>Redeems s0x shares back to their corresponding 0x assets.</td></tr></tbody></table>

#### Not Supported

* `A ↔ A` (direct underlying swaps)
* `0x ↔ 0x` (cross-synthetic swaps)

***

### 1) Entry Points & Previews

#### Swap functions

* **`swapExactIn(assetIn, assetOut, amountIn, receiver, referralCode)`**\
  Consume an exact `amountIn`; deliver the computed `amountOut`.
* **`swapExactOut(assetIn, assetOut, amountOut, maxAmountIn, receiver, referralCode)`**\
  Target an exact `amountOut`; compute and cap the required `amountIn` against `maxAmountIn`.

Both routes emit a canonical `Swap` event with in/out amounts and the referral code.

#### Deterministic previews (no state change)

* **`previewSwapExactIn(assetIn, assetOut, amountIn) → amountOut`**
* **`previewSwapExactOut(assetIn, assetOut, amountOut) → amountIn`**

Previews replicate settlement math, including redemption-fee snapshots for 0x→A and mint-side tin where applicable, so quote ≈ execution under identical state.

#### Conversions (utility)

* **`convertToAssets(asset, zeroXAmount)`**: 18-dec 0x → asset decimals (pure decimals normalization).
* **`convertToZeroXAssets(asset, assets)`**: asset decimals → 18-dec 0x (pure decimals normalization).

***

### 2) Pricing & Conversion Rules

#### A → 0x ([mint side](../../../user-flow-and-guides/how-to-buy-0xassets.md))

* **Pricing source:** oracle priced in the UCE instance’s **asset family** (BTC/ETH/USD).
  * If a per-asset **base feed** exists → use it.
  * Else **USD feed ÷ base/USD feed** (both freshness-checked via heartbeats).
* **Decimals normalization:** all 0x math at **18 decimals**; A assets are normalized into 18-dec 0x space.
* **Mint haircut (optional):** per-asset `mintHaircutBps` reduces 0x out defensively.
* **Tin (mint fee, optional):** per-asset `tinBps` reduces the user’s **net 0x**; an equivalent **0x amount is minted to the treasury**.

**Exact-in formula (conceptual):**

```solidity
stdIn      = normalizeTo18(assetIn, amountIn)
px         = priceInFamily1e18(assetIn)                 // oracle
zeroXPreTin   = stdIn * px / 1e18
zeroXAfterCut = zeroXPreTin * (1 - mintHaircutBps/1e4)
netOut     = zeroXAfterCut * (1 - tinBps/1e4)
feeZeroXTin   = zeroXAfterCut - netOut   // minted to treasury
```

**Exact-out:** computes the **gross pre-tin 0x** required to deliver `amountOut` and inverts the pipeline (including haircut) to find `amountIn`.

#### 0x → A (redemption side)

* **Pricing source:** **decimals normalization only** (no oracle).
* **Dynamic redemption fee:** charged **to the user** in 0x-terms, then normalized to A for treasury delivery.
  * **Rate snapshot** is taken for the quote and used for execution to preserve preview parity.
  * After settlement the base rate **decays hourly** and is **bumped** by the redeemed fraction of 0x supply, capped at **5%**.

**Exact-in formula (conceptual):**

```
feeRate    = redemptionRateSnapshot()         // 0 … 5% (1e18 scale)
feeZeroX   = zeroXIn * feeRate
grossU     = normalizeFrom18(assetOut, zeroXIn)
feeU       = normalizeFrom18(assetOut, feeZeroX)
netU       = grossU - feeU
```

**Exact-out:** compute `grossZeroX = underlyingToZeroX(assetOut, amountOut)` and add `feeZeroX = grossZeroX * feeRateSnapshot`.

#### 0x ↔ s0x ([ERC-4626](../../../user-flow-and-guides/how-to-buy-s0xasset.md))

* **0x → S0x:** `IERC4626(s).deposit(zeroXIn, receiver)` ⇒ shares minted per vault exchange rate.
* **S0x → 0x:** `IERC4626(s).redeem(sharesIn, address(this), address(this))` ⇒ 0x assets released.
* No UCE fee on these legs. Pricing is the ERC-4626 vault’s exchange rate.

***

### 3) Liquidity Sourcing & Delivery

#### Inbound A custody (A → 0x)

When A is received, UCE:

1. **Keeps a reserve slice** on-contract: `reserveBps` (default 25%) - fast liquidity for redemptions.
2. **Forwards the remainder to a pocket** chosen by routing (see[ Referral & Routing](swaps.md#id-4-referral-and-routing)).
   * Pockets are generic custody addresses; later **pulls are bounded by pocket balance and allowance**.

#### Outbound 0x delivery (A → 0x)

* **Protocol path** (no referral and caller not an allocator):
  1. Send **unreserved 0x on-hand**.
  2. **Pro-rata draw** from allocators’ `reservedZeroX` **capped by each allocator’s effective debt** (netting reduces their debt).
  3. **Mint any shortfall** to the receiver.
* **Allocator/referral path:** 0x is drawn from the **allocator’s `reservedZeroX`**; insufficient balance **reverts** (no silent socialization).

#### Outbound A delivery (0x → A)

* Deliver A to the receiver by pulling in order:
  1. **On-hand reserves** (consuming tracked reserved units first),
  2. **Referral allocator pocket** (if referral present),
  3. **Global pocket**.
* Pulls are strictly **bounded by allowance & balance**; if insufficient, the swap **reverts** (there is no “mint underlying” path).

***

### 4) Referral & Routing

* A **referral code** maps to a specific allocator.
* **A → 0x with referral:**
  * **Underlying** is forwarded to the **allocator’s pocket** (if configured; otherwise global pocket).
  * **0x** is delivered **from the allocator’s `reservedZeroX`** to the user.
  * If the allocator’s `reservedZeroX` < required amount, the swap **reverts** (enforces flow ownership).
* **0x → A with referral:** A delivery preferentially pulls from the **referral allocator’s pocket**; if insufficient, falls back to global context (still bounded by allowance/balance).

Allocators **cannot** perform 0x → A themselves; they must operate via underlying-side flows.

***

### 5) Exact-In vs Exact-Out Semantics

* **Exact-in** guarantees consumed input; the engine computes and enforces the precise output:
  * **A → 0x:** applies oracle pricing, mint haircut, and tin; then settles 0x outbound as per routing rules.
  * **0x → A:** snapshots redemption fee; delivers U using deterministic sourcing; routes the fee to treasury in A.
  * **0s ↔ s0x:** uses ERC-4626 `convertToShares/convertToAssets` paths to compute output.
* **Exact-out** guarantees delivered output; the engine computes the minimum input required:
  * **Caps input** with `maxAmountIn`; exceeding the cap reverts.
  * **Settlement parity checks** ensure the delivered amount matches the target; mismatches revert.

***

### 6) Fees: What, When, To Whom

* **Tin (mint fee)**: **A → 0x** only; reduces user’s 0x out and mints the fee in 0x to **treasury**.\
  Event: `TinFeeTaken(payer, assetIn, zeroXGrossBeforeTin, tinBps, feeZeroX, timestamp)`.
* **Dynamic redemption fee:** **0x → A** only; charged to user at a **snapshot rate**, delivered to **treasury in underlying**.\
  Event: `RedemptionFeeTaken(payer, assetOut, zeroXIn, feeRate, feeInUnderlying, timestamp)`.
* **No UCE fee** on **0x ↔ s0x**.

***

### 7) Pause, Safety, and Guard Rails

* **Global pause** and **per-asset pause** halt relevant swap legs gracefully.
* **Non-reentrancy** on state-changing flows.
* **Oracle guardrails** (heartbeat freshness, non-negative, non-stale) on A → 0x.
* **Pocket pulls bounded** by allowance & balance; insufficient pocket liquidity **reverts** rather than over-pulling.
* **Allocator restrictions:** 0x → A is disallowed for allocators; daily caps and ceilings constrain credit behavior on inventory creation (separate from swaps but relevant to settlement sources).

***

### 8) Failure & Edge-Case Handling

* **Unsupported pairs** → revert.
* **Zero amounts** or **zero receiver** → revert.
* **Asset paused** → revert.
* **Insufficient pocket allowance/balance** during A delivery → revert.
* [**Allocator**](allocators/#id-4.-repayment)**/referral 0x shortfall** in A → 0x → revert (flow ownership).
* **Exact-out exceeding `maxAmountIn`** → revert.
* **Settlement mismatch** (e.g., ERC-4626 returns unexpected shares/assets) → revert.

***

### 9) Practical Implications

* **Peg discipline:** Redemptions are oracle-free and capacity-bounded; a dynamic fee smooths surges and self-decays.
* **Capital efficiency:** Protocol uses unreserved 0x first; then nets **pro-rata** against allocators’ inventory (reducing their debt) before minting shortfall - minimizing new issuance.
* **Clear incentives:** Referral routes couple user inflows to allocator pockets and inventory; shortages are not socialized.
* **Deterministic quoting:** Previews match execution via rate snapshots and mirrored tin/decimals logic.

***

#### Quick Reference

* **Allowed pairs:** &#x41;_→0x, 0x→A, 0x→s0x, s0x→0x_.
  * **A→0x:** Oracle-priced, optional haircut, optional tin to treasury; reserves kept on-hand; remainder to pocket; 0x outbound = unreserved → pro-rata allocator inventory → mint.
  * **0x→A:** Decimals normalization; dynamic fee (snapshot) to treasury; A sourced = reserves → referral pocket → global pocket; no underlying mint.
  * **0x↔s0x:** ERC-4626 deposit/redeem; no UCE fee.
* **Referrals:** Attribute U inflows to allocator pockets and consume allocator `reservedZeroX`; shortages revert.
* **Safety:** Pair gating, pauses, heartbeats, allowance-bounded pulls, non-reentrancy.
