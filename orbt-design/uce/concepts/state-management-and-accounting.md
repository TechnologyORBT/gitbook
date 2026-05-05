# State Management and Accounting

This section describes how OrbtUCE represents system state on-chain and how that state evolves across swaps, credit events, reserves, and fees. The focus is on **who owns what**, **which counters move**, and **what invariants are preserved**—so operators, auditors, and integrators can reason about safety and solvency without reading code.

***

### 1) Core State Surfaces

#### Per-asset configuration (`assetCfg`)

Each supported underlying (U) has a unified record that drives liquidity, pricing, and control:

* **Pocket (custody address):** destination for the _non-reserve_ share of inbound U during **U→OX**. Later redemptions (**OX→U**) may pull from this pocket, but **only up to allowance and balance** (bounded pulls).
* **Family (BTC/ETH/USD):** ties the asset to the instance’s base unit for oracle math on **U→OX**.
* **Pause flag:** hard gate for any swap leg that touches the asset.
* **Tin (mint fee) bps:** applied on **U→OX**; reduces user OX out and mints the fee in OX to the treasury.
* **Reserve bps:** per-asset reserve ratio; fraction of inbound U kept on the UCE contract for instant redemptions.
* **Oracle config:** per-asset feeds (base or USD) with heartbeat staleness bounds and an optional **mint haircut**; used **only** on **U→OX** pricing.
* **`reservedUnderlying`:** on-chain counter of how much of that asset’s **on-hand** balance is earmarked as redemption reserve (grows on **U→OX**, shrinks when used for **OX→U**).

**Decimals cache:** UCE caches ERC-20 decimals (and fixes OX to 18) to make normalization **pure, deterministic, and gas-efficient** system-wide.

#### Global context

* **OX asset:** the instance’s synthetic base (18 decimals), minted/burned by the engine.
* **Treasury:** recipient of **tin** (mint-side fee, in OX) and **redemption fees** (in underlying).
* **Family-level oracle fallback:** base/USD feed used when an asset lacks a direct base feed.

#### Allocator state (`alloc`)

For each whitelisted allocator:

* **Credit line (`line`)** with **ceiling**, **dailyCap**, and UTC day counters for mint pacing.
* **Debt & inventory:**
  * **`reservedOx`**: allocator’s OX inventory held on the UCE and used to settle referred mints or protocol pro-rata draws.
  * **`baseDebt`**: liability recorded in **base units**; the **effective debt** is `baseDebt × debtIndex / 1e18`.
* **Borrow fee bps:** applied **on repay** in underlying (not at mint time).
* **Per-asset pockets:** optional allocator-specific custody for referral routing.
* **Referral code:** maps user flow to the allocator.
* **Linked-list pointers (`prev/next`)** and **`debtEpoch`** for ordering and epoch hygiene.

**Global credit aggregates:**

* **`debtIndex`** (init 1e18) scales all allocators’ `baseDebt` to effective debt.
* **`baseTotalDebt`** sums base debts across allocators (indexed by `debtIndex`).
* **`totalReservedOx`** totals all allocators’ OX inventories.

***

### 2) How State Evolves (Event-by-Event)

#### A) User mint — **U → OX**

1. **Custody split:** The engine retains **`reserveBps`** of `assetIn` on-contract (instant redemption capacity), increases that asset’s **`reservedUnderlying`**, and forwards the remainder to the selected **pocket** (global or referral allocator’s pocket).
2. **Pricing & fees:**
   * Oracle prices U in the instance’s family with staleness bounds; **optional haircut** reduces OX out.
   * **Tin** (if set) reduces user’s OX; the fee portion is **minted in OX to the treasury**.
3. **OX delivery (routing & sourcing):**
   * **Referral flow:** OX must come from the **allocator’s `reservedOx`**; insufficiency **reverts** (flow ownership).
   * **Protocol flow:** send **unreserved OX** first; if short, **pro-rata draw** from allocators’ `reservedOx` up to each allocator’s **available capacity** `min(reservedOx, effectiveDebt)`, _which reduces their debt_; then **mint** any residual shortfall.
   * Counters moved: `totalReservedOx` and each allocator’s `reservedOx` decrease when inventory is used; `baseDebt` decreases in base units for pro-rata applications.

**Invariants preserved:**

* **No oracle on redemption side** is touched; pricing risk is confined to the mint side.
* Pocket pulls remain **bounded by allowance & balance**; no hidden rehypothecation.

#### B) User redeem — **OX → U**

1. **Fee snapshot & quote parity:** Capture the **current (decayed) redemption rate**; compute fee in **OX** and normalize to **U** for treasury.
2. **Liquidity sourcing:** deliver underlying to the receiver using **on-hand reserve → referral pocket → global pocket**. `reservedUnderlying` for that asset is **consumed first** from on-hand.
3. **Post-trade dynamics:** update the dynamic fee by **decayed rate + redeemed fraction of OX supply**, capped (e.g., 5%).
4. **Accounting:** No allocator debt scaling; no underlying mint; treasury receives **fee in underlying**.

**Invariants preserved:**

* Redemptions are **oracle-free** and **capacity-bounded**; failure to source within bounds **reverts**.

#### C) Allocator credit mint — **create inventory & debt**

* **Who can call:** allocator themselves or ADMIN; allocator must be allowed and have a credit line.
* **Effects:**
  * Mint OX to UCE custody; increase allocator **`reservedOx`** and **`totalReservedOx`**.
  * Add **`baseDelta = amount × 1e18 / debtIndex`** to `baseDebt` and `baseTotalDebt`.
  * Enforce **dailyCap** (UTC bucket) and **ceiling** (on **effective** debt).
  * **Rebalance up** in the linked list to reflect larger effective debt.

#### D) Allocator repay — **reduce debt with underlying**

* **Inputs:** any supported **underlying** (repay in OX is disallowed).
* **Flow:** pull underlying; take **borrow fee** to treasury in underlying; convert principal to **OX-equivalent by decimals only**; cap at current **effective** debt; reduce `baseDebt` by `repayOx × 1e18 / debtIndex`; decrement `baseTotalDebt`; **rebalance down** in the list.
* **Side effect:** the received underlying remains **on-hand**, immediately strengthening **OX→U** capacity. Allocator **`reservedOx`** is not touched by repay.

***

### 3) Pro-Rata Engine & Linked-List Ordering

* **Why a list:** OrbtUCE maintains a light **doubly-linked list** of allocators, **approximately ordered by effective debt**.
* **When used:** On **protocol U→OX** if unreserved OX is insufficient, the engine does a **near-linear pass** to compute the denominator (= sum of `min(reservedOx, effectiveDebt)`) and assigns each allocator’s draw **pro-rata**, reducing `reservedOx` and **baseDebt** (via `debtIndex`).
* **Rebalancing:**
  * Credit mints **bubble up**; repayments **bubble down**.
  * Ordering need not be perfect; it only needs to keep heavy participants near the head to make pro-rata assignment efficient and fair without heap costs.

**Outcome:** Debt reduction is **index-consistent** and **gas-predictable**; inventory use **automatically nets** allocator liabilities.

***

### 4) Fees, Reserves, and Counters (Where Numbers Land)

* **Tin (U→OX):** deducted from user OX; **minted in OX** to **treasury**.
* **Redemption fee (OX→U):** charged in OX, **paid to treasury in underlying** at the decimals-normalized amount.
* **Reserves:** Per-asset **`reservedUnderlying`** rises on **U→OX** and falls when used to pay **OX→U**; the actual on-contract balance provides **auditable capacity**.
* **Totals:**
  * **`totalReservedOx`**: sum of allocators’ inventories; falls when inventory is consumed for settlement.
  * **`baseTotalDebt`** and **`debtIndex`**: global credit posture; index changes reprice **all** base debts proportionally **without looping**.

***

### 5) Controls, Safety & Determinism

* **Pausing:** global and per-asset pause enforce safe halts.
* **Bounded pulls:** all pocket withdrawals are capped by **allowance & balance**—redemptions revert rather than “borrow” liquidity.
* **Oracle scope:** **only** on **U→OX**; guarded by heartbeat integrity and non-stale reads.
* **Previews:** quote functions mirror settlement math (including redemption-rate snapshot and tin), so **preview ≈ execution** under unchanged state.
* **Epoch hygiene:** allocators carry a `debtEpoch`; if stale against `wipeEpoch`, the contract synchronizes the allocator before accepting new credit—preventing ghost state from affecting ceilings or pro-rata.

***

#### Practical Read-Out

* To **assess solvency** for redemptions, check **on-contract balances** plus **allowance-bounded** pocket capacity per asset; do not count unreachable pocket funds.
* To **understand credit risk**, inspect **`baseTotalDebt`**, **`debtIndex`**, and the distribution of **`reservedOx`** across allocators; remember that protocol pro-rata use **nets debt**.
* To **tune behavior**, adjust **reserve bps**, **tin**, **mint haircut**, credit **ceilings/daily caps**, and **dynamic redemption fee** parameters—each with a clear, isolated effect on the above counters and flows
