---
hidden: true
---

# UCE - Tech Dive

The Unified Collateral Engine (UCE) is the core liquidity and asset management layer of the ORBT protocol. It functions as a Peg Stability Module (PSM) that enables seamless, low-slippage swaps between underlying collateral assets (e.g., WBTC, cbBTC) and their corresponding synthetic 0x assets (e.g., 0xBTC). The UCE serves as the primary interface for users, allocators, and integrators to interact with the ORBT ecosystem.

Unlike traditional PSMs that simply maintain nearly 1:1 backing, the UCE implements an advanced credit system that allows whitelisted allocators to mint synthetic assets against credit lines, deploy capital into yield strategies, and manage their own liquidity pools. This design enables capital efficiency while maintaining robust collateralization and risk management.

{% hint style="danger" %}
Practically it is not advisable to compose a 1:1 backing model for 0xAsset <> Underlying since there can be slight or even major disparity among the family indexed price of each asset. Meaning that, it is true that stETH theoretically stays pegged to ETH generally but there could be instances where stETH might stay at a major delta from ETH price. Considering this, we have introduced oracles to always make sure the output or input of 0xAssets for issuance and redemption is catered using the supported underlying asset's oracle price and converted in terms of the family's index.&#x20;
{% endhint %}

The UCE is built around several key innovations:

* **Multi-Collateral Support**: A single UCE instance can manage multiple underlying assets within the same asset family (e.g., WBTC, cbBTC, tBTC for the BTC family) backing a single synthetic asset (e.g., 0xBTC for the BTC family)
* **Allocator Credit System**: Whitelisted allocators can mint 0x assets up to pre-approved credit lines with daily caps, ceiling limits and borrow fees but cannot access it until underlying has been deposited.
* **Referral Attribution**: Users can swap using allocator referral codes, directing underlying collateral to specific allocator pockets.
* **Dynamic Redemption Fees:** Fee rates increase with redemption pressure and decay over time, providing economic incentives for balanced liquidity.
* **Lazy Debt Accounting:** Proportional debt reduction across all allocators without iterating through allocator lists, enabling gas-efficient scaling.
* **Reserve Policy:** Configurable reserve ratios that keep a percentage of deposited collateral on-hand for instant redemptions.
* **Vault Integration:** Integration with s0xAsset which ERC-4626 for yield-bearing collateral management for swaps between 0x <> s0x as per current exchange rate.&#x20;

The UCE is designed for production deployment on Ethereum mainnet and is optimized for gas efficiency, security, and capital efficiency. It supports pausability, role-based governance controls, and comprehensive governance integration.

{% hint style="success" %}
OrbtUCE is **UUPS upgradeable proxy** contract
{% endhint %}

***

## Asset Family

Each UCE instance is deployed for a specific asset family denoted by this enum:&#x20;

<pre class="language-solidity"><code class="lang-solidity"><strong>enum {
</strong><strong>    BTC, // Bitcoin-backed assets (WBTC, cbBTC, tBTC)
</strong>    ETH, // Ethereum-backed assets (WETH, stETH, rETH)
    USD // USD stablecoin-backed assets (USDC, USDT, DAI)
}
</code></pre>

Orbt is not limited to these families, upon successful development, the model is scalable to multiple other families like SOL, RWA, GOLD etc.&#x20;

***

## Architecture

### Core Components

Data structures:

AllocatorState

```solidity
struct AllocatorState {
        bool allowed;
        uint256 baseDebt; // outstanding debt in base units (scaled by debtIndex)
        uint256 reservedOx; // allocator-reserved 0x inventory
        uint16 borrowFeeBps; // upfront borrow fee in bps
        LineOfCredit line; // allocator credit line
        mapping(address asset => address pocket) pocket;                // per-asset allocator pocket
        uint256 referralCode; // attribution code for user-initiated swaps
        uint256 debtEpoch; // epoch of this allocator's debt (masked if < wipeEpoch)
        address prev; // previous node in the allocator list
        address next; // next node in the allocator list
}
```

LineOfCredit

```solidity
struct LineOfCredit {
    uint128 ceiling; // Max total outstanding debt
    uint128 dailyCap; // Max mintable per 24h UTC window
    uint128 mintedToday; // Amount minted in current window
    uint32 lastMintDay; // Day index for daily cap reset
}
```

***

## Concepts

### Pair Model & What’s Supported

**Only these swap pairs are allowed (pair-gated):**

* **U → OX** _(Underlying to 0x asset)_
* **OX → U** _(0x asset to underlying)_
* **OX → S** _(0x asset to ERC-4626 s-asset wrapping the OX)_
* **S → OX** _(s-asset back to OX)_

**Not supported:** U ↔ U, OX ↔ OX.\
Enforced by `_validatePair` and `Errors.OrbtUCE__UnsupportedSwap()`

### Swaps

#### 1) Underlying → 0x Asset (U → OX)

**User flow**\
User deposits an underlying (e.g., WBTC) → receives 0x asset (e.g., 0xBTC).

**Entry points**

* `swapExactIn(assetIn=U, assetOut=OX, amountIn, receiver, referralCode)`
* `swapExactOut(assetIn=U, assetOut=OX, amountOut, maxAmountIn, receiver, referralCode)`
* Previews: `previewSwapExactIn`, `previewSwapExactOut`

**Pricing & math**

* **Uses oracle pricing** (`_toOxAmount`) in the engine’s **family base** (BTC/ETH/USD) with:
  * **Decimals normalization** (all OX math at 18 decimals).
  * **Per-asset mint haircut** (`mintHaircutBps`) if configured.
* **Mint fee (“tin”) may apply**: Admin can set `tinBps` per underlying via `setAssetTinBps`.
  * Tin **reduces user OX out** and the **fee is minted in OX to the treasury**.
  * Exact-in path: `_applyTinExactIn` (returns `(netOut, feeOx)`), fee emitted via `TinFeeTaken`.
  * Exact-out path grosses-up pre-tin with `_grossUpForTinExactOut`.

**Reserve split & routing**

* Engine keeps **`reserveBps`** (default **25%**) of inbound underlying on-hand; the remainder is **transferred** to the **selected pocket**:
  * Pocket selection via `_resolvePocket(sender, assetIn, referralCode)`:
    * Allocator caller → their per-asset pocket if set; else global.
    * Non-allocator with valid referral → allocator’s pocket if set; else global.
    * Otherwise → global pocket.
* **Note:** In this implementation pockets are **generic addresses** (not assumed ERC-4626). UCE **transfers** to pocket and later **pulls** from it subject to **allowance and balance**. (No `IERC4626.deposit()` is called for pockets in this version.)

**Outbound OX settlement (what funds your OX)**

* Priority for **U→OX with `referralCode == 0` and non-allocator caller**:
  1. **Unreserved** OX on-hand.
  2. **Pro-rata draw** from allocators’ **reservedOx** (capped by each allocator’s current effective debt).
  3. **Mint shortfall** to user.
* Otherwise (allocator caller or referral) we **settle from the allocator’s `reservedOx`**; if on-hand OX is insufficient to physically transfer, the shortfall is minted to the receiver.\
  (See `_settleOxOutbound`, `_settleOxOutbound_UToOxProtocol`, `_proRataDrawFromAllocators`.)

**Key points**

* **Fees:** **Tin** may apply (configurable). There is **no dynamic redemption fee** on U→OX.
* **Price:** Oracle-based (family base), optional **mint haircut**.
* **Reserves:** Default **25% on-hand**, remainder to pocket.
* **Referrals:** Route underlying to allocator’s pocket and consume allocator inventory if used.

***

#### 2) 0x Asset → Underlying (OX → U)

**User flow**\
User provides OX → receives underlying (e.g., WBTC).

**Entry points**

* `swapExactIn(assetIn=OX, assetOut=U, amountIn, receiver, referralCode)`\
  (User sends OX exact-in.)
* `swapExactOut(assetIn=OX, assetOut=U, amountOut, maxAmountIn, receiver, referralCode)`\
  (User targets exact underlying out; pays OX + fee.)

**Fees (dynamic redemption fee)**

* A **dynamic, decaying fee** is charged **to the user** on OX→U:
  * **Current rate** is a decayed value of `baseRedemptionRate` (`getCurrentRedemptionRate()` / `_currentRedemptionRate()`).
  * After each redemption, rate bumps by **redeemed fraction of total OX supply**, capped at **`MAX_REDEMPTION_RATE = 5%`**; it **decays hourly** (\~0.5% decay factor per hour) via `_rpow(DECAY_CONSTANT, hours, 1e18)`.
  * Fee is accounted in OX terms and **converted to underlying** for treasury in settlement events (`RedemptionFeeTaken`).
  * **Preview parity:** OX→U **previews take a rate snapshot** to match execution.

**Liquidity sourcing**

* Underlying is delivered from:
  1. **On-hand** balance (consumes tracked `reservedUnderlying` when present).
  2. **Referral allocator pocket** if a valid referral was supplied.
  3. **Global pocket** otherwise, via **pull bounded by pocket balance & allowance**.
* **There is no “mint new underlying” path** in this implementation.

**Other notes**

* **OX is not burned** on OX→U; UCE behaves PSM-style (holds OX on contract).
* **Allocators cannot perform OX→U**: they must use the **underlying** direction (`Errors.OrbtUCE__AllocatorMustSwapUnderlying()`).

***

#### 3) OX ↔ S (staking wrapper)

**User flows**

* **OX → S**: User stakes OX into an **ERC-4626 vault** whose `asset()` is the OX.
* **S → OX**: User redeems s-shares back to OX.

**Entry points**

* OX → S:
  * `swapExactIn(assetIn=OX, assetOut=S, amountIn, receiver, referralCode)` → `IERC4626(assetOut).deposit(amountIn, receiver)`
  * `swapExactOut(assetIn=OX, assetOut=S, amountOut, maxAmountIn, receiver, referralCode)` → deposit must yield exact shares.
* S → OX:
  * `swapExactIn(assetIn=S, assetOut=OX, amountIn, receiver, referralCode)` → `IERC4626(assetIn).redeem(amountIn, address(this), address(this))`
  * `swapExactOut(assetIn=S, assetOut=OX, amountOut, maxAmountIn, receiver, referralCode)` → uses `convertToShares/convertToAssets` previews.

**Key points**

* **No fees** on OX↔S in this implementation.
* **Exchange rate** governed by the s-vault (standard ERC-4626 mechanics).

### Allocators

**What is an allocator?**\
Whitelisted addresses (protocols, institutions, DAOs) with **credit lines** to mint OX **without immediate collateral**. They:

* Provide **inventory** (`reservedOx`) for referral-attributed swaps.
* Hold **pockets** for underlying assets per asset type.
* Accrue **debt** (scaled by a global `debtIndex`) when minting OX inventory.

#### Lifecycle

1. **Onboarding (Admin/Governance)**
   * Configure with `setAllocatorSingleByAdmin(init, assets, pockets, op)` or via governance `executeGovernanceAction(ACT_SET_ALLOCATOR, payload)`.
   * Fields include:
     * `allowed`
     * `line` = `{ceiling, dailyCap, mintedToday, lastMintDay}`
     * `borrowFeeBps`
     * Optional **allocator pockets** per asset.
   * A **referral code** is (re)generated on `SET_ALLOCATOR` or `UPDATE_REFERRAL_CODE`.
2. **Credit Minting**
   * `allocatorCreditMint(allocator, amount)` (callable by the **allocator** themselves or **ADMIN**):
     * Checks `allowed`, `line.dailyCap`, `line.ceiling` vs **effective debt**.
     * Mints OX **to the UCE**, increments `reservedOx`, and increases allocator **`baseDebt`** by `amount / debtIndex`.
     * Emits `CreditMinted`.
3. **Deployment / Usage**
   * Allocator deploys their **reservedOx** and sets **per-asset pockets** to receive underlying from referrals.
   * Users swapping with the allocator’s referral code send **underlying** to the allocator’s pocket while receiving OX from allocator’s `reservedOx`.
4. **Repayment**
   * `allocatorRepay(asset, assets)`:
     * Pulls **underlying** from allocator to UCE.
     * Takes **borrow fee in underlying**: `feeUnderlying = assets * borrowFeeBps / 1e4`, transfers fee to **treasury**.
     * Converts **principal underlying** to OX-equivalent via pure **decimals normalization** (no oracle) and caps to current **effective debt**.
     * Reduces `baseDebt` (scaled by `debtIndex`), updates totals & linked list order.
     * Emits `AllocatorRepaid`.

{% hint style="danger" %}
Allocator can mint 0xAssets into the UCE contract on a credit basis and hence will accrue fee until repaid. Unless a user (bringing allocator referral code) or the allocator brings supported underlying collateral, the credit cannot be accessed and hence stays out of circulation unless backed equally by required collateral.&#x20;
{% endhint %}

#### Roles, Privileges, Restrictions

* **Responsibilities**
  * Maintain `reservedOx` to serve referral flows.
  * Keep pockets funded/approved for UCE pulls on redemptions.
  * Observe `dailyCap`/`ceiling` and repay debt as required.
* **Privileges**
  * Mint OX up to credit **ceiling** without upfront collateral (`allocatorCreditMint`).
  * Receive referred underlying **directly** into per-asset pockets.
  * Configure own pockets through admin/governance flows.
* **Restrictions & Guards**
  * **Cannot do OX→U** swaps (`AllocatorMustSwapUnderlying`).
  * Must respect **daily cap** and **ceiling**; otherwise calls revert.
  * **Borrow fee** applies on **repayment** (in underlying), configurable per allocator.

***

### Referral Attribution & Inventory Use

**Referral codes** are deterministic:

```solidity
referralCode = uint256(keccak256(abi.encode(allocator, ceiling, dailyCap, borrowFeeBps)));
```

* Changing any of these **rotates** the expected referral code; governance/admin helpers update mappings:
  * Old code unmapped (if owned), new code mapped (if free); see `_applyAllocatorUpdate`.

**Swap flow with referral (U→OX):**

* Underlying **routes to allocator’s pocket** (if set).
* OX **comes from allocator’s `reservedOx`** to the receiver.
* If allocator balance insufficient → **revert** (for allocator/referral path), or fallback to protocol path if zero referral.

**Benefits**

* **Allocators** capture underlying deposits from referred users and can deploy them.
* **Users** get identical UX to standard swaps (fees, if any, same as U→OX rules).

***

### Dynamic Redemption Fee (OX → U)

* State:
  * `baseRedemptionRate` (scaled `1e18`)
  * `lastRedemptionTime`
  * Constants:
    * `MAX_REDEMPTION_RATE = 0.05e18` (5%)
    * Hourly exponential decay via `DECAY_CONSTANT ≈ 0.995e18` (≈0.5% decay factor/hour)
* Mechanics:
  1. **Preview uses a snapshot** of current rate to keep preview = execution.
  2. Fee in OX is **converted to underlying** and routed to **treasury** (`RedemptionFeeTaken`).
  3. After settlement, `_onRedemptionExecuted(oxRedeemed)`:
     * Decays old rate, then **adds `oxRedeemed / totalSupply`** (bounded, floored at 0, capped at max).
     * Updates `lastRedemptionTime`.
* Implications:
  * **Normal** periods → near-zero fee.
  * **High redemption pressure** → fee rises to protect liquidity, then **decays hourly** back down.

***

### Reserves, Pockets & Liquidity Pulls

* **Reserves**
  * Default `RESERVE_BPS = 2,500` (25%).\
    `setAssetReserveBps(asset, bps)` can override per-asset.
  * Tracked per-asset on-contract via `assetCfg[asset].reservedUnderlying` (increases on U→OX in-hand portion; consumed when used for OX→U deliveries).
* **Pockets**
  * **Global** pocket per asset: `assetCfg[asset].pocket`.
  * **Allocator** pockets by asset: `alloc[allocator].pocket[asset]`.
  * **Migration**: Admin/governance can update pockets; UCE will **migrate** up to **min(balance, allowance)** from old to new, and emits:
    * Global: `PocketSet(asset, oldPocket, newPocket, amountTransferred)` (emitted also by `addAsset` with `oldPocket = address(0)`).
    * Allocator: `AllocatorPocketSet(...)`.
* **Liquidity pulling** (on redemptions/admin withdraws):
  * Always bounded by **pocket token balance** and **pocket’s allowance** to the UCE (`Errors.OrbtUCE__InsufficientPocketLiquidity()`).
* **Admin liquidity ops**
  * `deposit(asset, receiver, assets)` — **admin-only**, custody forwarder; **no OX minted** (always returns 0). Transfers all to pocket.
  * `withdraw(asset, receiver, maxAssets)` — **admin-only**, pulls up to **min(balance, allowance)** from pocket and sends to `receiver`.
* **Emergency**
  * `emergencyWithdraw(asset, amount)` → pull from pocket (bounded) to **treasury**.

***

### Debt Accounting

* **Indexing**
  * Global `debtIndex` (starts at **`1e18`**).\
    Effective debt = `baseDebt * debtIndex / 1e18`.
* **Totals**
  * `baseTotalDebt` tracks the sum of base debts for the **current `wipeEpoch`**.
  * `allocatorDebt(a)` returns **0** if allocator’s `debtEpoch` is stale vs `wipeEpoch`; otherwise scales base by `debtIndex`.
* **Ordering**
  * A lightweight **linked list** (`headAllocator`/`tailAllocator`) approximates ordering by effective debt; maintained on mint/repay to support **pro-rata** draws in protocol U→OX settlements.

***

### Backing & Safety Invariants&#x20;

* **Supply vs Backing**
  * Total OX supply is expected to be backed by:
    * On-hand **underlying** + pullable from **pockets** (subject to allowance) + **effective allocator debt**.
* **Reserved inventory**
  * `totalReservedOx` ≤ OX on UCE contract.
* **Debt consistency**
  * Sum of allocator debts should equal `totalAllocatorDebt()` (scaled by index).

_(These are conceptual invariants derived from the code’s accounting model and pull constraints.)_

***

### Adding Assets & Oracles

* **Add asset**
  * `addAsset(asset, family, pocket, reserveBps)`
    * `family` **must match** this UCE instance’s `contractFamily`.
    * Caches `decimals`.
    * Emits **`PocketSet(asset, address(0), pocket, 0)`** (there is **no** `AssetAdded` event in this version).
* **Set 0x asset**
  * `setOxAsset(address)`: caches 18 decimals for the OX token and emits `OxAssetsSet`.
* **Oracles**
  * Per-asset configuration via `setOracle(asset, baseFeed, usdFeed, heartbeat, mintHaircutBps, enabled)`.
  * If `contractFamily != USD` and `baseFeed` missing, engine can derive from `usdFeed` and **global** `baseUsdFeed` (must be set via `setBaseUsdFeed`).
  * **Staleness** enforced by **heartbeat**; bad/stale data reverts.
  * **U→OX** pricing uses these oracles; **OX→U** is **decimals-only** (no oracle).

***

### Admin & Governance Quick Reference

* **Pause controls**
  * `pause()`, `unpause()`
  * `pauseAsset(asset)`, `unpauseAsset(asset)`
* **Pockets**
  * Global: `setPocket(asset, newPocket)` (migrates allowance-limited balance)
  * Allocator (governance payload): `ACT_SET_ALLOCATOR_POCKETS`
* **Allocator ops (admin helper)**
  * `setAllocatorSingleByAdmin(init, assets, pockets, op)` where `op` ∈:
    * `SET_ALLOCATOR`, `UPDATE_ALLOWED`, `UPDATE_REFERRAL_CODE`, `UPDATE_POCKET`, `UPDATE_LINE`, `UPDATE_BORROW_FEE`
* **Treasury**
  * `setTreasury(address)`
* **Tin (mint fee on U→OX)**
  * `setAssetTinBps(asset, bps)` (applies on U→OX; fee minted in OX to treasury)

***

### Important Behavior Changes vs Earlier Doc (Fixes)

* **U→OX is not 1:1 by decimals**: it is **oracle-priced** in the engine’s family base and may include a **mint haircut** and **tin fee** (if configured).\
  &#xNAN;_(Earlier doc incorrectly stated “No price slippage (1:1 adjusted for decimals)” and “No fees on minting”.)_
* **OX→U charges a dynamic fee** (decays hourly, capped at 5%), **paid by users**, routed to treasury in **underlying** terms.\
  &#xNAN;_(Earlier doc said fees only on redemption—correct—but missed the updated decay math and snapshot semantics.)_
* **Pockets are generic addresses in this version**: UCE **transfers** to pockets and **pulls** from them subject to **allowance & balance**; it does **not** `IERC4626.deposit()` for pockets.\
  &#xNAN;_(Earlier doc assumed ERC-4626 pockets.)_
* **No “mint underlying” fallback** on redemption: OX→U is sourced from **on-hand** + **pockets** (referral/global).\
  &#xNAN;_(Earlier doc mentioned a “minting path” or “debt reduction” on redemption of underlying—this does not exist.)_
* **Allocators are explicitly blocked from OX→U** swaps and must operate via underlying direction where required.
* **Tin fee exists for U→OX** (mint side), configurable per asset; **fee is minted in OX to the treasury**.

***

### Minimal Code Anchors (for readers)

* **Pair gating & swap router**: `_validatePair`, `swapExactIn`, `swapExactOut`
* **U→OX pricing**: `_toOxAmount`, `_fromOxAmount`, `_priceAssetInBase1e18`, `_readFeed`
* **Tin fee**: `setAssetTinBps`, `_applyTinExactIn`, `_grossUpForTinExactOut`, `TinFeeTaken`
* **Dynamic redemption fee**: `_currentRedemptionRate`, `_onRedemptionExecuted`, `_decayBaseRedemptionRate`, `RedemptionFeeTaken`
* **Reserves & pockets**: `_pullAssetWithReserveToPocket`, `_pushAssetFromContext`, `_pullFromPocket`, `PocketSet`, `AllocatorPocketSet`
* **Allocator credit & repay**: `allocatorCreditMint`, `allocatorRepay`, `AllocatorRepaid`, `CreditMinted`
* **Referral mapping**: `_generateReferralCode`, `_applyAllocatorUpdate`, `AllocatorReferralSet`
* **Debt & ordering**: `debtIndex`, `allocatorDebt`, `totalAllocatorDebt`, `_rebalanceAllocatorUp/_Down`, `_proRataDrawFromAllocators`

***

#### TL;DR

* **U→OX:** Oracle-priced, optional **mint haircut** + optional **tin fee** to treasury, reserves kept on-hand (default 25%), remainder to pocket; referral routes underlying to allocator pocket and consumes allocator inventory.
* **OX→U:** **Dynamic decaying fee** charged to user, sourced from on-hand + pockets; **no underlying mint** path.
* **OX↔S:** ERC-4626 deposit/redeem; no UCE fee.
* **Allocators:** Credit-mint OX into `reservedOx`, serve referrals, repay in underlying (fee in underlying to treasury), cannot swap OX→U.
* **Pockets:** Generic pull/push addresses with allowance gating (not ERC-4626 in this version).
