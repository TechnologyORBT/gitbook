# 0xAsset Credit Minting

This mechanism lets an approved **allocator** create 0x asset (**OX**) inventory on the UCE while recording a corresponding **debt** against their credit line. It is the primary way allocators provision OX that will later be used for **referral mints** and **pro-rata protocol draws**.

### Effects

When `allocatorCreditMint(allocator, amount)` succeeds, the engine:

* **Mints OX to the UCE contract** (not to the allocator) and credits the allocator’s **`reservedOx`** by `amount`.
* **Accrues allocator debt** by increasing `baseDebt` with `baseDelta = amount / debtIndex` (RAY scale), so **effective debt = baseDebt × debtIndex / 1e18**.
* **Tracks throughput** by incrementing `mintedToday` (UTC-bucketed) and rolling this counter when the day index changes.
* **Reorders** the allocator in the internal **linked list** (via `_rebalanceAllocatorUp`) to reflect higher effective debt—used later for efficient **pro-rata** draws.
* Updates global tallies: **`totalReservedOx += amount`** and **`baseTotalDebt += baseDelta`**.
* Emits **`CreditMinted(allocator, amount)`**.

> Note: Borrow fees are **not** charged at mint time; they apply on **repay** in underlying.
