# Debt Scaling (proportional socialization onto issuers)

### Mechanism & When It Triggers

**Debt scaling** in UCE is the protocol’s way to _automatically_ net down allocator debts **proportionally** when **protocol (**[**non-referred**](../uce/concepts/allocators/referral-vs-non-referral.md)**) A→0x mints** require 0x liquidity. If the user doesn’t provide a referral code (or the caller isn’t an allocator), UCE settles the 0x leg in three steps:

1. **Unreserved 0x on-hand** is used first.
2.  [**Pro-rata**](../uce/architecture.md#pro-rata-inventory-draw-and-linked-list-ordering) **allocator draw** kicks in for any remaining amount: each indebted allocator contributes from its **reserved 0x** up to its **available capacity** and receives a matching **debt reduction**.<br>

    $$available_i = min(reserved0x_i , effectiveDebt_i)$$<br>
3. **Mint shortfall**: if there’s still a remainder after the pro-rata draw, UCE mints the difference.

This flow _never_ touches an allocator on a [**referred**](../uce/concepts/allocators/referral-vs-non-referral.md) mint: referred A→0x **must** settle entirely from the referrer’s `reservedZeroX` (or the tx reverts). Socialization only applies to **protocol** A→0x.

***

### Accounting Model & Fairness Guarantees

*   **Index-scaled debts:** Each allocator’s debt is tracked in **base units** and scaled by a global **`debtIndex`** to an **effective debt**:\
    \
    $$\displaystyle \text{effectiveDebt}_i = \frac{\textstyle (\text{baseDebt}_i \times \text{debtIndex})}{10^{18}}$$<br>

    This model keeps per-allocator mutations O(1) and enables proportional adjustments without iterating every account for re-scaling.
* **Pro-rata selection set:** The engine computes a denominator  $$D = \sum_i \text{available}_i$$ and assigns each allocator a share ≈ $$availablei​/D$$  of the remaining 0x need. If D=0 (no eligible inventory), no draw occurs and the engine proceeds to mint.
* **Bounded by risk:** For each allocator , the engine caps the draw by both **inventory** and **debt** (`min(reservedZeroX, effectiveDebt)`), ensuring:
  * It never consumes more 0x than the allocator actually warehoused.
  * It never nets debt below zero from pro-rata operations.
* **Netting down debt:** The 0x taken from an allocator is treated as **repayment in 0x-equivalent**:
  * Decrease `reservedZeroX_i` and `totalReservedZeroX`.
  * Convert the applied amount to **base units** via `debtIndex` and subtract from `baseDebt_i` and `baseTotalDebt`.
  * Update the **linked list order** (approximate by effective debt) via `_rebalanceAllocatorDown` so the pro-rata traversal remains efficient and directionally fair under churn.
* **No slippage, user-neutral:** Users see deterministic fills (no pricing slippage introduced by socialization). The mechanism reallocates _issuer liabilities_, not user outcomes.

**Edge handling**

* If the total available inventory D is **less than** the required amount, the engine draws all D pro-rata and **mints the rest**.
* If D is **greater than** or **equal to** the need, the engine draws exactly the target amount in a single pass (with a follow-up top-off pass to close rounding gaps), preserving proportionality.

**Design outcomes**

* **Alignment:** Issuers who warehouse more inventory and carry more debt shoulder a proportionate share of protocol flow, _and receive_ proportionate **debt reduction**.
* **Safety:** The engine never “borrows” from pockets; redemptions of underlying remain bounded by **on-hand + allowance-bounded pocket liquidity**.
* **Extensibility:** The index-based model (`baseDebt` × `debtIndex`) supports future _global_ proportional adjustments (e.g., governance-driven write-downs) without touching every account, while today’s live socialization happens through the **pro-rata 0x draw** path described above.
