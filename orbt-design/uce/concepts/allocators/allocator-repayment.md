# Allocator Repayment

This routine reduces an allocator’s effective debt by accepting **underlying assets** and updating system state in a debt-indexed, gas-predictable way.

### Purpose

* **Decrease allocator liability**: Convert an on-chain transfer of underlying into an 0x-equivalent principal and apply it against the allocator’s **effective debt** (scaled by `debtIndex`).
* **Fund redemptions**: The underlying received remains **on the** [**UCE**](../../) **contract** (on-hand), increasing immediate redemption capacity for 0x→A flows.

### Operational Flow

1. **Access / sanity checks**
   * Caller must be a **whitelisted allocator**.
   * **Paused** state blocks execution.
   * `assets > 0`, `asset` must be a supported underlying (repay in 0x is **disallowed**).
2. **Pull funds**
   * UCE pulls `assets` of the specified underlying **from the allocator** into **UCE custody** (kept on-hand; not forwarded to pockets).
3. **Borrow fee (if configured)**
   * Compute $$\displaystyle \text{feeUnderlying} = \text{assets} \times \frac{\text{borrowFeeBps}}{10^{4}}$$.
   * Transfer the fee to **`treasury`** (reverts if fee > 0 and treasury is unset).
   * Remaining **principal**: $$principalUnderlying = assets - feeUnderlying$$.
   * If principal is zero, the function **returns** (no debt change).
4. **Normalization & cap**
   * Convert principal to **0x-equivalent** via **decimals normalization only** (no oracle):\
     $$0xEquivalent = normalizeTo18(asset, principalUnderlying)$$.
   * Cap repayment to current **effective debt** (cannot over-repay).
5. **Debt accounting (index-aware)**
   * Convert the 0X-equivalent repayment to **base units**:\
     $$\displaystyle \text{baseRepay} = \text{repay0x} \times \frac{10^{18}}{\text{debtIndex}}$$.
   * Reduce allocator’s `baseDebt` by `baseRepay` (capped at its current base).
   * Reduce **`baseTotalDebt`** accordingly.
   * **Rebalance down** the allocator in the linked list to reflect lower effective debt (improves fairness for future pro-rata operations).
6. **Event**
   * Emit $$AllocatorRepaid(repayer=allocator, allocator=allocator, amount=repay0x)$$ with the **0X-equivalent principal** applied.

***

### Effects & Invariants

* **On-hand liquidity up**: Underlying stays on the UCE contract, immediately usable for 0x→A settlement (step-1 source).
* **No oracle dependency**: Repayment uses **pure decimals normalization**, avoiding pricing risk on the liability side.
* **Index-consistent**: Using `debtIndex` preserves proportionality across allocators and keeps global adjustments **O(1)**.

***

### Guardrails / Reverts

* Caller is not an allocator → revert.
* `assets == 0` → revert.
* Unsupported/invalid `asset` or attempting to repay in 0x → revert.
* Fee > 0 but `treasury == address(0)` → revert.
* If current **effective debt is zero**, the function **returns early** (no-op) after checks.

***

### Practical Notes

* **Fee timing**: Borrow fee is charged **on repay** (not at credit mint), and is taken **in underlying** to treasury.
* **Capacity signaling**: Because repaid underlying stays on-hand, observers can see rising contract balances translating to stronger redemption depth without waiting for pocket pulls.
* **Concurrency safety**: `nonReentrant` + pause gates protect the path during volatile events.
