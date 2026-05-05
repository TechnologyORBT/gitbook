# 0xAsset Credit Minting

This mechanism lets an approved **allocator** create 0x asset (**0x**) inventory on the UCE while recording a corresponding **debt** against their credit line. It is the primary way allocators provision 0x that will later be used for **referral mints** and **pro-rata protocol draws**.

### Effects

When `allocatorCreditMint(allocator, amount)` succeeds, the engine:

* **Mints 0x to the UCE contract** (not to the allocator) and credits the allocator’s **`reservedZeroX`** by `amount`.
* **Accrues allocator debt** by increasing `baseDebt` with `baseDelta = amount / debtIndex` (RAY scale), so  $$\displaystyle \text{effective debt} = \text{baseDebt} \times \frac{\text{debtIndex}}{10^{18}}$$.
* **Tracks throughput** by incrementing `mintedToday` (UTC-bucketed) and rolling this counter when the day index changes.
* **Reorders** the allocator in the internal **linked list** (via `_rebalanceAllocatorUp`) to reflect higher effective debt - used later for efficient **pro-rata** draws.
* Updates global tallies: **`totalReservedZeroX += amount`** and **`baseTotalDebt += baseDelta`**.
* Emits **`CreditMinted(allocator, amount)`**.

> Note: Borrow fees are **not** charged at mint time; they apply on **repay** in underlying.
