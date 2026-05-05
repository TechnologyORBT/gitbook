# Referral Attribution and Inventory Consumption

UCE ties user flow to allocator responsibility through [**referral codes**](referral-vs-non-referral.md) and uses allocator inventory to settle mints and (when applicable) withdrawals in a way that is auditable and incentive-aligned.

* **Attribution via referral codes.** Each allocator has a deterministic **referral code** mapped on-chain to their address. When a user performs **A → 0x** with a valid referral:
  * The **underlying** is routed to the allocator’s **per-asset pocket**.
  * The user’s **0x** is delivered **from the allocator’s `reservedZeroX`** inventory.
  * If `reservedZeroX` is **insufficient**, the swap **reverts.** The allocator’s shortfall is not socialized to the protocol. This enforces **flow ownership**: allocators who attract flow must maintain inventory and pocket liquidity/allowance.
* **Inventory consumption on protocol mints (no referral).** For **A → 0x** without a referral (and when the caller is not an allocator), UCE delivers 0x in this order:
  1. **Unreserved 0x** on the UCE contract
  2. **Pro-rata draw** across allocators’ inventories up to each allocator’s **available capacity** `min(reservedZeroX, effectiveDebt)`, which **reduces their debt** (base-indexed) as inventory is consumed. The allocator set is maintained in a lightweight **linked list** approximately ordered by **effective debt**, enabling near-linear pro-rata assignment without expensive sorting.
  3. **Mint** any remaining shortfall.
* **Referral influence on redemptions.** For **0x → A** with a referral code, UCE **preferentially pulls underlying** from the **referral allocator’s pocket** before falling back to global context (all pulls remain bounded by pocket balance and allowance). Redemptions never mint underlying.&#x20;

**Result:** Referrals align incentives (flow ↔ inventory), keep settlement capacity explicit (pockets/allowances), and ensure that when the protocol taps allocator inventory on non-referred flow, it **repays their debt automatically**, producing fair, market-like netting without hidden subsidies.
