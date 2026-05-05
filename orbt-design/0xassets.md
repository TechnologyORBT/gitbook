# 0xAssets

### Overview

**0xAssets (0x)** are the protocol’s base-denominated ERC-20s (18 decimals) that abstract a collateral family (e.g., **0xBTC** for the BTC family, **0xETH** for the ETH family). Each [UCE ](uce/)instance manages exactly one 0x (its [**family base**](uce/concepts/swaps.md#instrument-classes)) and treats all supported underlyings in that family (e.g., WBTC/cbBTC for BTC; WETH/stETH-like wrappers for ETH) as A (underlying asset) against that 0x. A [yield wrapper](../protocol-mechanics/savings-rate.md#how-it-works-s0xassets-your-savings-token) of an 0x (e.g., **s0xBTC**, **s0xETH**) is an ERC-4626 vault (**s0x** **asset**) whose `asset()` is the 0x itself.

**Key properties**

* **ERC-20, 18 decimals, upgrade-safe minter/burner:** 0x has fixed 18-dec precision; UCE is the sole minter/burner/settler.
* **Deterministic** [**pair gating**](uce/concepts/swaps.md#swap-planes-pair-gated-conversion)**:** Only four swap planes exist: **A↔0x** and **0x↔s0x**. There is _no A_↔A or 0x↔0x path.
* **No oracle on** [**redemption**](system-overview.md#id-1-1-asset-model) **(0x→A):** Redemption is a pure **decimals normalization** (0x 18-dec → A native decimals), minus a [**dynamic redemption fee**](uce/concepts/dynamic-redemption-fees.md) that decays over time and bumps on usage.
* **Oracle on mint (A→0x):** Minting 0x from A uses Chainlink-style feeds (base or USD + base/USD) and applies an optional **mint haircut** (bps). Results are then reduced by optional **tin** (mint-side fee) that mints 0x to treasury.
* **Inventory semantics:** 0x on UCE can be (i) **unreserved** (protocol inventory) or (ii) **reserved** for allocators (per-allocator `reservedZeroX`). [Referral mints](uce/concepts/allocators/referral-vs-non-referral.md) must settle from the referrer’s reserved inventory.
* **Backed by custody & pockets:** Underlyings are held partly on UCE ([per-asset **reserve** slice](uce/concepts/reserve-policy.md)) and mostly in [**pockets**](pocket/) (global or allocator-specific custody/vaults) with allowance-bounded pulls.

**Examples**

* **0xBTC instance** (family = BTC):
  * _Underlying Asset (A):_ WBTC, cbBTC, …
  * _0x:_ 0xBTC (18-dec).
  * _s0x asset:_ s0xBTC (ERC-4626 with `asset() == 0xBTC`).
  * _A→0x path:_ WBTC → 0xBTC uses BTC-base pricing (WBTC/BTC or WBTC/USD + BTC/USD), optional haircut, tin to treasury.
  * _0x→A path:_ 0xBTC → WBTC is decimals-only scaling, minus dynamic redemption fee (in WBTC).
* **0xETH instance** (family = ETH):
  * _Underlying Asset (A):_ WETH, staked ETH variants mapped via policy.
  * _0x:_ 0xETH (18-dec).
  * _s0x asset:_ s0xETH (ERC-4626 with `asset() == 0xETH`).
  * _A→0x path:_ WETH → 0xETH via ETH-base pricing, optional haircut, tin.
  * _0x→A path:_ 0xETH → WETH via decimals-only scaling, minus redemption fee (in WETH).

***

### Mechanics & Policy (applies to 0xBTC/0xETH)

* [**Swaps**](uce/concepts/swaps.md)
  * **A→0x:** Pull A; retain per-asset **`reserveBps`**  n UCE; deposit remainder to the selected **pocket** (allocator’s pocket on referral, otherwise global). \
    $$Quote = oracle price × decimalsNormalization × (1 − haircut)$$. **Tin** (bps) is taken in 0x and minted to treasury. **Settlement source for 0x out:**
    * Referral: **must** use referrer’s `reservedZeroX` (reverts if insufficient).
    * No referral: unreserved protocol 0x → **pro-rata draw** from allocators’ available capacity (nets down their debt) → mint any shortfall.
  * **0x→A:** Pull 0x to UCE; compute **snapshot** redemption fee rate; deliver exact A using **on-hand reserve first**, then allowance-bounded pocket withdrawal (referral pocket when present, else global). Fee is transferred in A to treasury; post-settlement the rate **bumps** by redeemed fraction and **decays** multiplicatively over time.
  * **0x↔s0x:** Pure ERC-4626 accounting: `deposit/redeem/convertToShares/convertToAssets` with no oracle use; no protocol fee on these legs.
* **Decimals discipline**
  * 0x is always **18 decimals**. Conversions 0x↔A use integer scaling with cached token decimals; previews mirror execution.
* **Risk knobs**
  * **Per-asset `reserveBps`:** Higher improves immediate redemption capacity in that A; lower increases yield reliance on pockets.
  * **Tin (mint fee) & Mint Haircut:** Tin monetizes A-side issuance to treasury; haircut provides oracle-side safety margin on A→0x.
  * **Per-asset pause & oracle heartbeat:** Any leg for a paused asset reverts; stale/invalid oracle reverts A→0x.
* **Allocator coupling (referrals)**
  * Referral **routes A** to the allocator’s pocket and **requires** 0x delivery from that allocator’s `reservedZeroX`.
  * Non-referred flow can **socialize** via pro-rata 0x draw, which **reduces allocators’ debts** in index-consistent fashion.
* [**Invariants** ](uce/concepts/backing-invariants.md)**(intuitive)**
  * **No synthetic A minting:** A delivered is strictly bounded by **on-hand + allowance-bounded pocket liquidity**.
  * **Preview parity on redemption:** Snapshot fee ensures A out matches preview (absent external state changes).
  * **Index-consistent debt:** All allocator debt changes occur in **base units** and scale by global **debtIndex**; pro-rata 0x draws net debt down fairly.

**Takeaway:** 0xBTC and 0xETH are the [**family bases**](uce/concepts/swaps.md#instrument-classes) that make collateral fungible inside their domain, with oracle-guarded minting, oracle-free redemption, allocator-exact [referral routing](uce/architecture.md#referral-routing-flow-ownership), and deterministic settlement bounded by custody reality.
