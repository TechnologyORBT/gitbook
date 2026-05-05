# Referral vs Non-Referral

#### 1) What a Referral Code Does

* **Attribution & Routing**
  * **A→0x (issuance)**: Inbound **U** is routed to the **referrer’s Pocket**; **0x** to the user **must** come from the referrer’s **`reservedZeroX`**. If `reservedZeroX < required`, the tx **reverts** (prevents free riding on other allocators).
  * **0x→A (redemption)**: **A** is sourced from the **referrer’s Pocket** first (subject to allowance/balance). If insufficient allowance or funds, the tx **reverts** (e.g., `InsufficientPocketLiquidity`).
* **Non-referral**
  * **A→0x**: Uses protocol inventory, then **pro-rata** allocator inventory (reducing their effective debt), and only then mints shortfall per system rules.
  * **0x→A**: Uses **on-hand reserve** and **global Pocket** (allowance-bounded).

#### 2) Fees & Economics (Same Rates; Different Sources)

* **A→0x**: Oracle pricing (+ optional mint haircut) then `tinBps` applied **in 0x** to treasury. **Referral does not change `tinBps`**, only who supplies the 0x (referrer vs global).
* **0x→A**: **Dynamic redemption fee** (snapshotted at execution; preview-consistent) applied in **A**. **Referral does not change the fee curve**, only **which Pocket** funds the redemption.
* **0x↔s0x**: **No UCE fee**; conversion follows the ERC-4626 exchange rate of the savings vault.

#### 3) Invariants & Failure Modes

* **Referred A→0x must fully settle from the referrer’s `reservedZeroX`** → otherwise **revert**.
* **0x→A with referral** requires **referrer Pocket**:
  * **Allowance** from Pocket → UCE must be high enough.
  * **Balance** must cover the redemption net of fee.
* **Non-referral paths** must respect: asset pause gates, oracle freshness (A→0x), pair validity (A↔0x, 0x↔s0x only), and zero-amount guards.
* **Fees are preview-parity**: `previewSwapExactIn/Out` match execution because the fee **rate is snapshotted** at call start.

#### 4) Implementation Tips

* **Always preview** (`previewSwapExactIn/Out`) and pass the correct `referralCode` (`0` if none).
* For **referral** integrations, add **pre-tx checks**: Pocket **allowance** to UCE and **balance** thresholds.
* **Surface clear UX errors**: “Referrer’s reserved 0x insufficient”, “Pocket allowance too low”, “Asset paused”, “Oracle stale”.

#### 5) Minimal Examples (Pseudocode)

```solidity
// Referred A -> 0x (must use referrer’s reservedZeroX)
uint256 quote = UCE.previewSwapExactIn(A, zeroX, amtA);
IERC20(A).approve(address(UCE), amtA);
uint256 out = UCE.swapExactIn(A, zeroX, amtA, user, refCode); // reverts if reservedZeroX < out

// Referred 0x -> A (Pocket must have allowance+balance)
uint256 needA = 100_000e6;
uint256 zeroXIn = UCE.previewSwapExactOut(zeroX, A, needA);
IERC20(zeroX).approve(address(UCE), zeroXIn);
uint256 paid = UCE.swapExactOut(zeroX, A, needA, zeroXIn, user, refCode); // reverts if Pocket gating fails

// Non-referral A -> 0x (referralCode = 0)
uint256 zeroXOut = UCE.previewSwapExactIn(A, zeroX, amtA);
IERC20(A).approve(address(UCE), amtA);
uint256 got = UCE.swapExactIn(A, zeroX, amtA, user, 0);
```

See [**Swap API & Planes**](../../../../protocol-mechanics/integration/uce.md#id-2-swap-planes-and-how-to-integrate) in this document for exact function signatures, approvals, and recommended slippage guards.
