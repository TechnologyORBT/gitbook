# UCE

> This page documents **how to integrate swaps** with UCE for all three planes:
>
> * **Underlying Asset → 0xAssets** (issuance of 0xAssets from underlyings)\
>   abbreviation: **A → 0x**
> * **0xAssets → Underlying Asset** (redemption)\
>   abbreviation: **0x → A**
> * **0xAssets ↔ s0xAssets** (savings-share conversions via ERC-4626)\
>   abbreviation: **0x <-> s0x**
>
> It also covers **referral vs. non-referral** origination and **all fees** that can apply.

***

This guide covers swaps in the [Unified Collateral Engine (UCE)](../../orbt-design/uce/) across three paths: **Underlying Asset (A) ⇄ 0xAssets (0x)** for mint/redemption, and **0x ⇄ s0xAssets (s0x)** for ERC-4626 savings-share conversions. **0x** are 18-dec synthetic assets (e.g., 0xUSD, 0xBTC); **s0x** are vault shares whose value follows the ERC-4626 exchange rate.

[**Allocators**](../../orbt-design/allocator/) are permissioned liquidity partners; their[ **referral codes**](../../orbt-design/uce/concepts/allocators/referral-attribution-and-inventory-consumption.md) route A inflows to the allocator’s pocket and consume that allocator’s reserved 0x first. \
\
**Underlying Asset → 0x** uses oracle pricing (with optional mint haircut) and applies **tinBps** (fee in 0x to treasury). **0x → A** applies a **dynamic redemption fee** (snapshotted at execution; rises with pressure, decays over time).\
&#x20;**0x ⇄ s0x** is oracle-free and fee-free at UCE (yield comes from the vault’s rate).&#x20;

{% hint style="danger" %}
Always **preview** swaps for quotes/slippage, pass the appropriate **referralCode** (0 if none), and set **ERC-20 approvals** before calling `swapExactIn` / `swapExactOut`.
{% endhint %}

### 1) API

**Swap using ORBT's swapping interface**

```solidity
function swapExactIn(
  address assetIn,
  address assetOut,
  uint256 amountIn,
  address receiver,
  uint256 referralCode
) external returns (uint256 amountOut);

function swapExactOut(
  address assetIn,
  address assetOut,
  uint256 amountOut,
  uint256 maxAmountIn,
  address receiver,
  uint256 referralCode
) external returns (uint256 amountIn);
```

**Previews (recommended for UI & slippage control)**

```solidity
function previewSwapExactIn(address assetIn, address assetOut, uint256 amountIn)
  external view returns (uint256 amountOut);

function previewSwapExactOut(address assetIn, address assetOut, uint256 amountOut)
  external view returns (uint256 amountIn);
```

**Helpful read-only utilities**

```solidity
// Family base discovery & routing context
function assetFamilies(address asset) external view returns (AssetFamily); // e.g., BTC/ETH/USD
function pockets(address asset) external view returns (address);           // global pocket set for the asset

// Unit conversions (18-dec OX ↔ native decimals of A)
function convertToZeroXAssets(address asset, uint256 assets) external view returns (uint256);
function convertToAssets(address asset, uint256 numZeroXAssets) external view returns (uint256);

/// tinBps per underlying asset is stored in AssetCfg (exposed via events/config UIs)
/// dynamic redemption fee is time-varying; preview methods already account for it
```

**Events for analytics**

```solidity
event Swap(
           address indexed assetIn, 
           address indexed assetOut, 
           address sender,
           address indexed receiver, 
           uint256 amountIn, uint256 amountOut, 
           uint256 referralCode
);
```

**Token approvals you’ll need (per path)**

* **A → 0x**: `IERC20(assetIn).approve(UCE, amountIn)`
* **0x → A**: `IERC20(zeroX).approve(UCE, amountIn)` (for `swapExactIn`) or `maxAmountIn` (for `swapExactOut`)
* **0x → s0x**: `IERC20(zeroX).approve(UCE, amountIn)`
* **s0x → 0x**: `IERC20(s0x).approve(UCE, shares)` (shares are ERC-4626 shares)

***

### 2) Swap Planes & How To Integrate

#### A) Underlying Asset → 0x ([Issuance](../../user-flow-and-guides/how-to-buy-0xassets.md))

**What it does**

* Pulls A from user, splits by **reserveBps** (on-hand buffer vs. pocket), computes 0x via **oracle pricing** (+ optional mint haircut), then applies **tinBps** fee (0x minted to treasury). User receives **net 0x**.

**Fees**

* **`tinBps` (per-asset mint fee, in 0x):** Deducted from 0x out; minted to `treasury`.
* **No redemption fee here.**

[**Referral behavior**](../../orbt-design/uce/concepts/allocators/referral-vs-non-referral.md)

* **With referralCode ≠ 0**:
  * Inbound **A** is routed to the **allocator’s pocket**.
  * **0x out must come from the referrer’s `reservedZeroX`**. If insufficient, **tx reverts**.
* **Without referral (referralCode = 0)**:
  * 0x out settles from **unreserved protocol inventory**, then **pro-rata allocator inventory** (which _reduces their debt_), then **mints shortfall** if needed.

**How to call**

```solidity
// 1) Show quote to user
uint256 zeroXOut = UCE.previewSwapExactIn(A, zeroX, amountA);

// 2) Approve & swap
IERC20(A).approve(address(UCE), amountA);
uint256 received = UCE.swapExactIn(A, zeroX, amountA, user, referralCode);
```

**Notes**

* **Decimal safety**: UCE handles decimals normalization; 0x is always 18-decimals.
* **Oracle requirements**: If oracle is stale/disabled for the asset, the call reverts.
* **Asset pause**: If either side is paused, the call reverts.

***

#### B) 0x → Underlying Asset (Redemption)

**What it does**

* Pulls **0x** from user; computes **snapshot** [**redemption fee**](../../orbt-design/0xusd/redemption-rate-and-shortfall-socialization.md) **rate**; delivers A using on-hand reserve first, then **allowance-bounded pocket withdrawal** (referral pocket if provided, else global pocket). Transfers fee to treasury in **A** and **bumps** the dynamic redemption rate post-settlement.

**Fees**

* **Dynamic redemption fee (time-varying):**
  * **Preview-consistent**: UCE snapshots the fee rate at start; previews match execution.
  * Increases with redemption pressure (fraction of 0x redeemed vs. supply) and **decays over time**.

**Referral behavior**

* **With referralCode ≠ 0**: A is pulled from the **referrer’s pocket** (subject to pocket allowance & balance).
* **Without referral**: UCE uses **global context** (on-hand + global pocket).
  * If pocket allowance is insufficient, the call reverts (`InsufficientPocketLiquidity`).

**How to call**

```solidity
// Exact-in: user knows OX in
uint256 aOut = UCE.previewSwapExactIn(zeroX, A, zeroXIn); // net of current redemption fee (snapshotted on exec)
IERC20(zeroX).approve(address(UCE), oxIn);
uint256 receivedA = UCE.swapExactIn(zeroX, A, zeroXIn, user, referralCode);

// Exact-out: user wants specific A out
uint256 zeroXNeeded = UCE.previewSwapExactOut(zeroX, A, targetA); // includes fee at snapshot rate
IERC20(zeroX).approve(address(UCE), zeroXNeeded);
uint256 paidzeroX = UCE.swapExactOut(zeroX, A, targetA, zeroXNeeded, user, referralCode); // reverts if > max
```

**Notes**

* **Allocators cannot redeem 0x→A** (they must settle in underlying); allocator callers to 0x→A revert.
* **Pocket allowance**: Ensure the pocket (global or referrer’s) has approved UCE; otherwise redemption may fail.

***

#### C) 0x ↔ s0x (ERC-4626 [savings shares](../../user-flow-and-guides/how-to-buy-s0xasset.md))

**What it does**

* **0x → s0x**: UCE deposits 0x into the s-vault (`IERC4626(assetOut).deposit`) and mints **shares** (S) to receiver.
* **s0x → 0x**: UCE redeems shares (`IERC4626(assetIn).redeem`) for **0x** to receiver.
* This is **oracle-free** and **fee-free** at UCE level (vault may accrue yield via exchange rate).

**Fees**

* **No UCE fee on 0x → s0x.** Exchange rate is determined by the ERC-4626 vault.

**How to call**

```solidity
// Ox -> s0x (savings deposit)
uint256 shares = UCE.previewSwapExactIn(zeroX, S0X, zeroXIn);
IERC20(zeroX).approve(address(UCE), zeroXIn);
uint256 mintedShares = UCE.swapExactIn(zeroX, S0X, zeroXIn, user, 0);

// s0x -> Ox (savings withdrawal)
uint256 zeroXOut = UCE.previewSwapExactIn(S0X, zeroX, sharesIn);
IERC20(S0X).approve(address(UCE), sharesIn);
uint256 receivedZeroX = UCE.swapExactIn(S0X, zeroX, sharesIn, user, 0);
```

**Notes**

* For exact-out variants, use `previewSwapExactOut` and pass `maxAmountIn` guards.
* The **s0x → 0x exchange** depends on the **ERC-4626 exchange rate** (`convertToShares` / `convertToAssets`).

***

### 3) Referral vs Non-Referral&#x20;

**A. Behavior at a Glance**

* **Referral flow (`referralCode != 0`)**: A inflows route to the referrer’s **Pocket**; 0x outflows **must** be served from the referrer’s **reservedZeroX** (else revert). 0x→A pulls A **from the referrer’s Pocket** (subject to allowance/balance).
* **Non-referral (`referralCode = 0`)**: A inflows follow **global routing**; 0x outflows first use unreserved protocol inventory, then pro-rata allocator inventory, minting shortfalls only as designed. 0x→A pulls from **on-hand reserve + global Pocket**.

> See the full reference for edge cases, invariants, and examples: [**Referral vs Non-Referral**](../../orbt-design/uce/concepts/allocators/referral-vs-non-referral.md)

**B. Integration Checklist**

* Always pass the correct `referralCode` (use `0` if none).
* For **referral** redemptions, ensure the **referrer’s Pocket** maintains ERC-20 **allowance** to UCE and has sufficient **balance**.
* Fees: **A→0x** applies `tinBps` (in 0x) regardless of referral; **0x→A** applies the **dynamic redemption fee** equally, referral only affects **liquidity source**, not the fee math.

> Deep dive on routing, inventory consumption, and failure modes: [**Referral vs Non-Referral**](../../orbt-design/uce/concepts/allocators/referral-vs-non-referral.md)

| Origination       | A → 0x                                                                                                                                  | 0x → A                                                                       |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **With referral** | Underlying routed to **allocator pocket**; 0x **must come from allocator's `reservedZeroX`** (else revert).                             | Underlying pulled **from referrer’s pocket** (subject to allowance/balance). |
| **No referral**   | 0xAssets from **unreserved portion of 0xAssets in UCE by  protocol** → **pro-rata allocator draw deducting debt** → **mint shortfall**. | Underlying from **on-hand reserve**, then **global pocket**.                 |

***

### 4) Fees & Slippage Guards

* **A → 0x (mint)**:
  * **Oracle-priced** (per asset family), optional **mint haircut**, then **`tinBps`** (deducted in 0x, minted to treasury).
  * Use `previewSwapExactIn` / `previewSwapExactOut` for net 0x and required A.
* **0x → A (redeem)**:
  * **Dynamic redemption fee** (time-varying). Snapshotted at swap start → **preview parity**.
  * Use `previewSwapExactIn` (0x in → A out) or `previewSwapExactOut` (A out → 0x in).
* **0x ↔ s0x**: No UCE fee; ERC-4626 exchange rate applies.

**Recommended UI guards**

* For **Exact-In**: display `previewSwapExactIn` and allow a **minOut** tolerance on the client side.
* For **Exact-Out**: compute `previewSwapExactOut` and set **maxAmountIn = preview** (or small buffer).

***

### 5) Common Reverts & Integration Checks

* **Paused asset**: `assetIn` or `assetOut` paused → revert.
* **Invalid pair**: Only **A↔0x** and **0x↔s0x** are supported (no A↔A, no 0x↔0x).
* **Zero amounts / bad receiver**: `amountIn == 0`, `amountOut == 0`, or `receiver == address(0)` → revert.
* **Allocator restrictions**: Allocators cannot call **0x→A** (must use underlying).
* **Oracle issues (A→0x)**: Stale/disabled feeds or family mismatch → revert.
* **Pocket liquidity/allowance** (**0x→A**): If allowance/balance on pocket is insufficient, revert (integrators should ensure pockets keep allowances open).
* **Referral invariants**: Referred **A→0x** must be fully served by referrer’s `reservedZeroX` (or it reverts).

***

### 6) Minimal End-to-End Examples

**Mint 0x from A (no referral)**

```solidity
uint256 quote = UCE.previewSwapExactIn(A, zeroX, amountA);
IERC20(A).approve(address(UCE), amountA);
uint256 zeroXOut = UCE.swapExactIn(A, zeroX, amountA, msg.sender, 0);
```

**Redeem 0x to A (exact-out with referral)**

```solidity
uint256 zeroXNeeded = UCE.previewSwapExactOut(zeroX, A, wantA);
IERC20(zeroX).approve(address(UCE), zeroXNeeded);
uint256 paid = UCE.swapExactOut(zeroX, A, wantA, zeroXNeeded, msg.sender, referralCode);
```

**Deposit 0x into savings: ERC-4626 (get s0x)**

```solidity
uint256 shares = UCE.previewSwapExactIn(zeroX, s0x, zeroXIn);
IERC20(zeroX).approve(address(UCE), zeroXIn);
uint256 minted = UCE.swapExactIn(zeroX, S0X, zeroXIn, msg.sender, 0);
```

**Redeem s0x back to 0x (exact-out)**

```solidity
uint256 sharesIn = UCE.previewSwapExactOut(s0x, zeroX, needZeroX);
IERC20(S0X).approve(address(UCE), sharesIn);
uint256 spentShares = UCE.swapExactOut(sZeroXAsset, zeroX, needZeroX, sharesIn, msg.sender, 0);
```

***

### 7) Implementation Tips

* **Always preview** before calling swap; reflect **tin** and **redemption fee** in the UI.
* **Set appropriate allowances** for ERC-20s and ERC-4626 shares.
* **Handle referral codes**: pass `0` for non-referred flow; pass on-chain mapped code for referred users.
* **Surface failures clearly**: oracle stale, pocket allowance low, paused asset, invalid pair.
* **Block explorer & analytics**: index the `Swap` event for volumes, referral share, and path usage.
