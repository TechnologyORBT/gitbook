---
description: >-
  This page explains the financial architecture of OrbtUCE and the design
  choices behind peg stability, liquidity routing, and allocator credit.
---

# Architecture

<figure><img src="../../.gitbook/assets/12 - UCE.png" alt=""><figcaption></figcaption></figure>

### Design Objectives

1. **Hard Peg Behavior for Redemptions**
   * Users swapping **0x→A** should experience deterministic execution at **decimals-normalized parity minus a** [**dynamic fee**](concepts/dynamic-redemption-fees.md) (no oracle dependency on this leg).
   * Liquidity for redemptions comes from [**on-hand reserves**](concepts/reserve-policy.md) and [**pullable pockets**](../pocket/reserve-buffer-and-rebalancing.md); we do not “print” underlying.
2. **Elastic Minting with Risk Controls**
   * Users mint **A→0x** at **oracle-priced** parity in the instance’s asset family with optional **mint haircut** and **tin** (mint fee).
   * The system’s exposure comes from [_allocators_ ](../allocator/what-is-an-allocator.md)_-_ credit-minted 0x inventory - bounded by ceilings and daily caps.
3. **Capital Efficiency Without Global Loops**
   * Debt is tracked via a **global debt index**, giving **O(1)** proportionality for all allocators; we avoid walking N allocators for every global change.
4. **Predictable Liquidity Sourcing**
   * Liquidity is split between [**on-contract reserves**](concepts/swaps.md#id-3-liquidity-sourcing-and-delivery) and [**pockets**](../pocket/what-is-a-pocket.md) (custody addresses). Pulls are **bounded by allowance** to make solvency auditable on-chain.
5. **Stability Under Stress**
   * A **dynamic, decaying redemption fee** throttles 0x→A during surges, then self-resets as pressure subsides.

### The Peg & Liquidity Model

#### Two-Layer Liquidity

* **Layer 1:** [**On-hand reserves**](concepts/swaps.md#id-3-liquidity-sourcing-and-delivery)\
  A fixed portion of every A→0x remains on the UCE. This is fast liquidity for normal 0x→A traffic.
* **Layer 2:** [**Pockets** ](../pocket/what-is-a-pocket.md)**(custody addresses)**\
  The remainder is forwarded to a **global pocket** or, for referral flows, to the **allocator’s pocket**. Pockets are generic addresses; all pulls are bounded by **pocket balance and allowance**. This makes redemption capacity measurable and prevents implicit rehypothecation.

**Why this split?**

* Higher reserves → faster redemptions, lower yield.
* Lower reserves → higher yield, more pocket pulls and gas.

#### Redemption Determinism

*   **0x→A** is **decimals-normalized** (no oracle). Users get:

    $$A_{out} = (0x_{in} \times 10^{(A_{decimals} - 18)}) - dynamicFee_{inA}$$
* Liquidity is sourced in order: **reserves → referral allocator pocket (if provided) → global pocket**. If pockets lack allowance/balance, the transaction reverts, no hidden minting of underlying.

***

### Pricing & Fees (Economic Rationale)

#### Mint Side (A→0x)

* **Oracle-priced** in the [instance’s family](concepts/swaps.md#instrument-classes) (BTC/ETH/USD), optionally **haircut** to protect against oracle drift or asset idiosyncrasies.
* **Tin (mint fee)** reduces user 0x out and is **minted as 0x to the treasury**.\
  &#xNAN;_&#x52;ationale:_ mint side is where supply expands; we collect proto-seigniorage to fund operations and build a defensive buffer while keeping redemptions as clean as possible.

#### Redemption Side (0x→A)

* [**Dynamic redemption fee**](concepts/dynamic-redemption-fees.md) increases with **recent redemption volume** (as a fraction of 0x supply), capped at **5%**, and **decays hourly** back toward zero.
* **Preview parity:** we **snapshot** the rate at quote time so <kbd>preview == execution</kbd>.\
  &#xNAN;_&#x52;ationale:_ when redemptions spike, we raise marginal cost to slow the drain, giving pockets time to refill and avoiding race-to-the-exit dynamics. When pressure falls, fee decays automatically - no governance intervention needed.

***

### Allocator Credit Architecture

#### Why Allocators?

[Allocators ](concepts/allocators/)are professional liquidity managers. We let them **credit-mint 0x** (inventory) within strict [limits](concepts/allocators/debt-accounting-with-debt-index.md), then **absorb user A inflows** via [referral routing](concepts/allocators/referral-vs-non-referral.md). This aligns usage (user flow) with responsibility (inventory & pockets) without burdening end users.

#### Credit Discipline

* **Ceiling (stock limit)** caps total _effective_ debt per allocator.
* **Daily Cap (flow limit)** paces issuance day-by-day (UTC buckets).
* **Borrow Fee (underlying, on repayment)** aligns allocator behavior with system costs.

#### Debt Indexing (Systemic Efficiency)

We track each allocator’s debt in **base units** and scale them by a global **`debtIndex`**:

$$
effectiveDebt[a] = baseDebt[a] * debtIndex
$$

* If we need a **global proportional adjustment** (e.g., system-wide debt reduction or pro-rata netting), we change **one scalar** instead of touching N allocators.
* This delivers **O(1)** updates for system events and keeps per-allocator actions **O(1)** as well.

***

### Pro-Rata Inventory Draw & Linked-List Ordering

#### When We Draw Pro-Rata

For **protocol A→0x** flow ([no referral](concepts/allocators/referral-vs-non-referral.md), user is not an allocator):

1. Use **unreserved** 0x on-hand.
2. If still short, **draw pro-rata from allocators’ `reservedZeroX`**, **capped by each allocator’s** [**effective debt**](concepts/allocators/debt-accounting-with-debt-index.md) (you cannot draw more than they owe).
3. If a shortfall remains, **mint** the remainder to the user.

This makes allocator inventory a **soft buffer for the protocol**, and when used, it **reduces their debt** automatically - clean netting with no cross-subsidies.

#### Why the Linked List?

We maintain a lightweight **linked list** of allocators, **approximately ordered by effective debt**. It serves two purposes:

1. **Pro-Rata Math with Fewer Touches**
   * We iterate the list once (or a small number of passes) to:
     * Compute the denominator = `sum of`` `_`available`_` ``pro-rata capacity (min(reservedOx, effectiveDebt))` across allocators.
     * Assign each allocator’s share = $$need * available / denominator$$
   * If rounding leaves a residual, we do a short second pass to allocate leftovers.\
     This is **near-linear in the number of allocators that matter** (those with capacity), and the list is small and stable in practice.
2. **Stable Priority Under Changes**
   * On **mint** (debt up) and **repay** (debt down), we **bubble** allocators toward head/tail. We don’t need perfect ordering - only that “larger debt” allocators tend to be encountered earlier when calculating shares. This keeps the pro-rata assignment well-behaved without costly re-heaps or sorts.

**Why not a heap or full sort?**

* We don’t need exact ordering for pro-rata; **approximate ordering suffices**.
* A heap introduces more pointer churn and higher gas to maintain strict invariants.
* The linked list gives **cheap, monotone adjustments** and **predictable gas** on inserts/moves.

**Economic effect:**

* Allocators with more debt and more `reservedZeroX` get proportionally more of the protocol draw, which **automatically reduces** their debt.
* This encourages allocators to **keep inventory funded** and **stay active**; idle allocators neither help nor free-ride.

***

### Referral Routing (Flow Ownership)

* A [**referral code**](concepts/allocators/referral-vs-non-referral.md) maps user A→0x flows to an **allocator**.
* In those swaps:
  * The **underlying** goes to the allocator’s **pocket** (their custody).
  * The user’s **0x** is delivered **from the allocator’s `reservedZeroX`** (must be sufficient).
* If allocator inventory is insufficient, the **referral swap reverts** (we do not silently fall back to protocol inventory).\
  &#xNAN;_&#x44;esign intent:_ Ownership of flow comes with responsibility to hold inventory and maintain pullable pockets. It avoids socializing an allocator’s shortage across the system.

***

### Oracle Scope & Failure Containment

* **Oracles are only used on the mint side (A→0x)**. This limits oracle risk to **supply expansion** moments; **redemptions are oracle-free** (decimals only).
* We enforce **heartbeats** and revert on stale or bad prices.
* For non-USD families, if an asset has only a USD feed, we compose with a **base/USD** feed; otherwise, a direct **base feed** is preferred.\
  &#xNAN;_&#x52;esult:_ A bad oracle can only impair **new mints**; it cannot block **redemptions**, preserving user exit even under oracle stress.

***

### Stability Under Stress (How the System Self-Stabilizes)

1. **Redemption Surge → Fee Rises**\
   The [dynamic fee](concepts/dynamic-redemption-fees.md) increases with redemption share of supply, raising marginal cost to slow depletion of reserves and pockets.
2. **Allocator Buffer → Pro-Rata Draw**\
   For protocol mints, if unreserved 0x is short, we _first_ net against allocator inventory pro-rata, **reducing their debt** and improving the system’s balance sheet.
3. **Deterministic Capacity → Pocket Constraints**\
   Redemptions are bounded by demonstrable, pullable liquidity; this resists “black box” insolvency and forces allocators to keep pockets topped and approved.
4. **Decay Back to Normal**\
   When pressure subsides, the redemption fee **decays hourly,** so pricing reverts near parity without governance action.

***

### Governance & Parameterization (What to Tune and Why)

* **ReserveBps (per asset)**\
  Trade-off between redemption latency and capital efficiency. Raise during periods of persistent redemptions; lower when liquidity is consistently slack.
* **TinBps (per asset)**\
  Captures value on supply expansion. Useful to fund operations, bootstrap treasury, and offset oracle risk. Keep low for strong product-market fit; raise when mint demand is opportunistic.
* **Mint Haircut (per asset)**\
  Defensive bias for mint pricing in volatile or thinly fed assets; reduces over-issuance risk if oracles lag.
* **Ceiling & DailyCap (per allocator)**\
  Risk-weighted exposure per allocator. Daily caps deter burst issuance that could strain redemption capacity.
* **Borrow Fee (per allocator)**\
  Aligns allocator repayment incentives with treasury funding; adjust by allocator profile and track record.
* **Dynamic Fee Parameters**\
  Cap (max rate) and decay profile can be tuned to your ecosystem’s rhythm: faster decay for active, deep markets; slower decay if refills are lumpy.

***

### Typical Scenarios

1. **Calm Market**
   * Minting dominates: tin slowly funds treasury; reserves stay near target; redemption fee \~0%.
2. **Volatility Spike**
   * Users redeem 0x→A; fee ramps to slow the outflow.
   * Allocators with pockets approve more allowance; reserves and pockets satisfy orderly exits.
3. **Refill & Recovery**
   * Underlying flows back in (either via users or allocator activity); redemption fee decays.
   * Protocol mints may net against allocator inventory, **reducing allocator debts** without manual coordination.

***

### Why This Architecture Works

* **User-first exits**: deterministic 0x→A, no oracle risk, bounded by visible liquidity.
* **Allocator-aligned**: inventory used benefits them (debt down), but shortages are **not socialized**.
* **Operationally simple**: debt index, linked-list ordering, and pocket allowances keep gas low and behavior predictable.
* **Governance-light**: dynamic fee decays and reserve splits let the system breathe without constant parameter fiddling.

***

#### TL;DR

* **Peg stability** comes from **oracle-free redemptions**, **bounded liquidity pulls**, and a **dynamic, decaying redemption fee**.
* **Credit scalability** comes from **debt indexing** and **pro-rata inventory netting** guided by a **linked list** for near-linear passes.
* **Flow ownership** via [**referrals** ](concepts/allocators/referral-vs-non-referral.md)ensures allocators who win flow must also maintain inventory and pockets - keeping incentives tight and the system solvent.

