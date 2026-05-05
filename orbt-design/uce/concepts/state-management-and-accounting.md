# State Management and Accounting

This section describes how OrbtUCE represents system state on-chain and how that state evolves across swaps, credit events, reserves, and fees. The focus is on **who owns what**, **which counters move**, and **what invariants are preserved -** so operators, auditors, and integrators can reason about safety and solvency without reading code.

***

### 1) Core State Surfaces

#### Per-asset configuration (`assetCfg`)

Each supported underlying (U) has a unified record that drives liquidity, pricing, and control:

* **Pocket (custody address):** destination for the _non-reserve_ share of inbound A during **A→0x**. Later redemptions (**0x→A**) may pull from this pocket, but **only up to allowance and balance** (bounded pulls).
* **Family (BTC/ETH/USD):** ties the asset to the instance’s base unit for oracle math on **A→0x**.
* **Pause flag:** hard gate for any swap leg that touches the asset.
* **Tin (mint fee) bps:** applied on **A→0x**; reduces user 0x out and mints the fee in 0x to the treasury.
* **Reserve bps:** per-asset reserve ratio; fraction of inbound A kept on the UCE contract for instant redemptions.
* **Oracle config:** per-asset feeds (base or USD) with heartbeat staleness bounds and an optional **mint haircut**; used **only** on **A→0x** pricing.
* **`reservedUnderlying`:** on-chain counter of how much of that asset’s **on-hand** balance is earmarked as redemption reserve (grows on **A→0x**, shrinks when used for **0x→A**).

**Decimals cache:** UCE caches ERC-20 decimals (and fixes 0x to 18) to make normalization **pure, deterministic, and gas-efficient** system-wide.

#### Global context

* **0x asset:** the instance’s synthetic base (18 decimals), minted/burned by the engine.
* **Treasury:** recipient of **tin** (mint-side fee, in 0x) and **redemption fees** (in underlying).
* **Family-level oracle fallback:** base/USD feed used when an asset lacks a direct base feed.

#### Allocator state (`alloc`)

For each whitelisted allocator:

* **Credit line (`line`)** with **ceiling**, **dailyCap**, and UTC day counters for mint pacing.
* **Debt & inventory:**
  * **`reservedZeroX`**: allocator’s 0x inventory held on the UCE and used to settle referred mints or protocol pro-rata draws.
  * **`baseDebt`**: liability recorded in **base units**; the **effective debt** is `baseDebt × debtIndex / 1e18`.
* **Borrow fee bps:** applied **on repay** in underlying (not at mint time).
* **Per-asset pockets:** optional allocator-specific custody for referral routing.
* **Referral code:** maps user flow to the allocator.
* **Linked-list pointers (`prev/next`)** and **`debtEpoch`** for ordering and epoch hygiene.

**Global credit aggregates:**

* **`debtIndex`** (init 1e18) scales all allocators’ `baseDebt` to effective debt.
* **`baseTotalDebt`** sums base debts across allocators (indexed by `debtIndex`).
* **`totalReservedZeroX`** totals all allocators’ 0x inventories.

***

### 2) How State Evolves (Event-by-Event)

#### A) User [mint](../../../user-flow-and-guides/how-to-buy-0xassets.md): A **→ 0x**

1. **Custody split:** The engine retains **`reserveBps`** of `assetIn` on-contract (instant redemption capacity), increases that asset’s **`reservedUnderlying`**, and forwards the remainder to the selected **pocket** (global or referral allocator’s pocket).
2. **Pricing & fees:**
   * Oracle prices A in the instance’s family with staleness bounds; **optional haircut** reduces 0x out.
   * **Tin** (if set) reduces user’s 0x; the fee portion is **minted in 0x to the treasury**.
3. **0x delivery (routing & sourcing):**
   * **Referral flow:** 0x must come from the **allocator’s `reservedZeroX`**; insufficiency **reverts** (flow ownership).
   * **Protocol flow:** send **unreserved 0x** first; if short, **pro-rata draw** from allocators’ `reservedZeroX` up to each allocator’s **available capacity** `min(reservedZeroX, effectiveDebt)`, _which reduces their debt_; then **mint** any residual shortfall.
   * Counters moved: `totalReservedZeroX` and each allocator’s `reservedZeroX` decrease when inventory is used; `baseDebt` decreases in base units for pro-rata applications.

**Invariants preserved:**

* **No oracle on redemption side** is touched; pricing risk is confined to the mint side.
* Pocket pulls remain **bounded by allowance & balance**; no hidden rehypothecation.

#### B) User redeem: 0x **→ A**

1. **Fee snapshot & quote parity:** Capture the **current (decayed) redemption rate**; compute fee in **0x** and normalize to **A** for treasury.
2. **Liquidity sourcing:** deliver underlying to the receiver using **on-hand reserve → referral pocket → global pocket**. `reservedUnderlying` for that asset is **consumed first** from on-hand.
3. **Post-trade dynamics:** update the dynamic fee by **decayed rate + redeemed fraction of 0x supply**, capped (e.g., 5%).
4. **Accounting:** No allocator debt scaling; no underlying mint; treasury receives **fee in underlying**.

**Invariants preserved:**

* Redemptions are **oracle-free** and **capacity-bounded**; failure to source within bounds **reverts**.

#### C) Allocator credit mint: **create inventory & debt**

* **Who can call:** allocator themselves or ADMIN; allocator must be allowed and have a credit line.
* **Effects:**
  * Mint 0X to UCE custody; increase allocator **`reservedZeroX`** and **`totalReservedZeroX`**.
  * Add **`baseDelta = amount × 1e18 / debtIndex`** to `baseDebt` and `baseTotalDebt`.
  * Enforce **dailyCap** (UTC bucket) and **ceiling** (on **effective** debt).
  * **Rebalance up** in the linked list to reflect larger effective debt.

#### D) Allocator repay: **reduce debt with underlying**

* **Inputs:** any supported **underlying** (repay in 0x is disallowed).
* **Flow:** pull underlying; take **borrow fee** to treasury in underlying; convert principal to **0x-equivalent by decimals only**; cap at current **effective** debt; reduce `baseDebt` by `repayZeroX × 1e18 / debtIndex`; decrement `baseTotalDebt`; **rebalance down** in the list.
* **Side effect:** the received underlying remains **on-hand**, immediately strengthening **0x→A** capacity. Allocator **`reservedZeroX`** is not touched by repay.

***

### 3) Pro-Rata Engine & Linked-List Ordering

* **Why a list:** OrbtUCE maintains a light **doubly-linked list** of allocators, **approximately ordered by effective debt**.
* **When used:** On **protocol A→0x** if unreserved 0x is insufficient, the engine does a **near-linear pass** to compute the denominator (= sum of `min(reservedZeroX, effectiveDebt)`) and assigns each allocator’s draw **pro-rata**, reducing `reservedZeroX` and **baseDebt** (via `debtIndex`).
* **Rebalancing:**
  * Credit mints **bubble up**; repayments **bubble down**.
  * Ordering need not be perfect; it only needs to keep heavy participants near the head to make pro-rata assignment efficient and fair without heap costs.

**Outcome:** Debt reduction is **index-consistent** and **gas-predictable**; inventory use **automatically nets** allocator liabilities.

***

### 4) Fees, Reserves, and Counters (Where Numbers Land)

* **Tin (A→0x):** deducted from user 0x; **minted in 0x** to **treasury**.
* **Redemption fee (0x→A):** charged in 0x, **paid to treasury in underlying** at the decimals-normalized amount.
* **Reserves:** Per-asset **`reservedUnderlying`** rises on **A→0x** and falls when used to pay **0x→A**; the actual on-contract balance provides **auditable capacity**.
* **Totals:**
  * **`totalReservedZeroX`**: sum of allocators’ inventories; falls when inventory is consumed for settlement.
  * **`baseTotalDebt`** and **`debtIndex`**: global credit posture; index changes reprice **all** base debts proportionally **without looping**.

***

### 5) Controls, Safety & Determinism

* **Pausing:** global and per-asset pause enforce safe halts.
* **Bounded pulls:** all pocket withdrawals are capped by **allowance & balance -** redemptions revert rather than “borrow” liquidity.
* **Oracle scope:** **only** on **A→0x**; guarded by heartbeat integrity and non-stale reads.
* **Previews:** quote functions mirror settlement math (including redemption-rate snapshot and tin), so **preview ≈ execution** under unchanged state.
* **Epoch hygiene:** allocators carry a `debtEpoch`; if stale against `wipeEpoch`, the contract synchronizes the allocator before accepting new credit - preventing ghost state from affecting ceilings or pro-rata.

***

#### Practical Read-Out

* To **assess solvency** for redemptions, check **on-contract balances** plus **allowance-bounded** pocket capacity per asset; do not count unreachable pocket funds.
* To **understand credit risk**, inspect **`baseTotalDebt`**, **`debtIndex`**, and the distribution of **`reservedZeroX`** across allocators; remember that protocol pro-rata use **nets debt**.
* To **tune behavior**, adjust **reserve bps**, **tin**, **mint haircut**, credit **ceilings/daily caps**, and **dynamic redemption fee** parameters - each with a clear, isolated effect on the above counters and flows
