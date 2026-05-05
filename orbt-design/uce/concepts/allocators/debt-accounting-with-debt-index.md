# Debt Accounting with Debt Index

UCE tracks allocator liabilities using a **base/Effective** decomposition that keeps accounting proportional, gas-predictable, and easy to reason about during system events.

*   **Debt index (global scalar):** `debtIndex` (init `1e18`) converts each allocator’s stored **`baseDebt`** into **effective debt**:

    ```
    effectiveDebt[a] = baseDebt[a] * debtIndex / 1e18
    ```

    All comparisons, ceilings, and pro-rata caps are evaluated in **effective** terms, while state mutations update **base** (so changes remain O(1)). Global tallies mirror this split via **`baseTotalDebt`**.
* **Crediting vs. repaying:**
  * **Credit mint** adds `baseDelta = amount * 1e18 / debtIndex` to the allocator’s `baseDebt` and increments `baseTotalDebt`; their **`reservedOx`** increases by `amount`.
  * **Repay (in underlying)** computes an **OX-equivalent principal** by decimals normalization (no oracle), caps to current **effective** debt, then reduces `baseDebt` by `baseRepay = repayOx * 1e18 / debtIndex` (and decrements `baseTotalDebt`).
  * This index-aware math ensures allocators remain **proportionally** comparable without iterating over peers.
* **Epoch hygiene:** Each allocator carries a `debtEpoch`; if it lags the global `wipeEpoch`, the allocator’s base accounting is synchronized before accepting new credit, preventing stale state from leaking into calculations.
* **Linked-list ordering for pro-rata operations:**\
  UCE maintains a lightweight **doubly-linked list** of allocators, approximately ordered by **effective debt**. On **credit mint** the allocator is **bubbled up** (`_rebalanceAllocatorUp`); on **repay** it is **bubbled down** (`_rebalanceAllocatorDown`).
  * During **protocol U→OX** settlement, if unreserved OX is insufficient, UCE computes pro-rata draws across allocators’ **available capacity** (`min(reservedOx, effectiveDebt)`) by **single-pass** traversals of this list, then applies reductions in base units.
  * The approximate ordering is sufficient for fair, near-linear pro-rata assignment while avoiding the gas overhead of strict heaps/sorts.

**Net effect:** The index keeps **system-wide proportionality** cheap and robust; the list makes **pro-rata netting** and settlement sourcing efficient, ensuring debt reduction and inventory consumption scale smoothly as participants grow.
