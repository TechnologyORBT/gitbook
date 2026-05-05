# Allocator

**Allocators** are permissioned market-makers and liquidity partners that interface directly with the Unified Collateral Engine (UCE). They \
\
(i) provision custody liquidity via **pockets**, \
(ii) warehouse **reserved 0x** inventory, and \
(iii) intermediate user flow through **referral attribution,**\
\
earning spread and carry while assuming operational and liquidity-readiness risk.

### **Core capabilities**

* [**Credit line**](../uce/concepts/allocators/0xasset-credit-minting.md) **(0x issuance):** Each allocator receives a **ceiling** (max effective debt) and **dailyCap** (issuance velocity). Calling credit mint increases **reserved 0x** and the allocator’s **baseDebt** (scaled by the global **debtIndex**).
* **Inventory & settlement:**
  * **Referral mints (A→0x)**: user supplies underlying with the allocator’s referral code → UCE routes the non-reserve portion to the allocator’s **pocket** and **must** settle user 0x **from that allocator’s reserved 0x** (if insufficient, the tx reverts - no socialization).
  * **Protocol mints (no referral)**: UCE uses unreserved 0x on-hand, then performs a **pro-rata draw** across allocators’ **available capacity** `min(reservedZeroX, effectiveDebt)`, which **reduces their debt** proportionally; any shortfall is minted.
* **Custody liquidity (pockets):** Per-asset custody addresses (EOA/multisig/vault) that hold the allocator’s underlying balances. UCE pulls from pockets **strictly within allowance and balance -** no implicit borrowing.
* **Yield & fees:** Allocators can deploy pocket balances to conservative venues (e.g., ERC-4626 vaults) and monetize primary market flow. Borrowing cost is realized via **borrow fee (bps) at repay**; mint-side user fee (**tin**) goes to the protocol treasury in 0x; **redemption fees** (during 0x→A) are paid in underlying to the treasury and do not alter allocator debt.
* [**Risk boundaries**](../0xusd/allocator-incentives-and-risk.md)
  * **Liquidity obligation:** Keep pockets funded and allowances high to ensure fast redemptions.
  * **Inventory obligation:** Maintain **reserved 0x** to serve referred flow without triggering reverts.
  * **Credit discipline:** Issuance respects **ceiling** and **dailyCap** at the contract level (hard reverts on breach).
  * **System coupling:** If protocol needs 0x for non-referred mints, **pro-rata consumption** of allocator inventory nets down allocator debt via the **debtIndex -** aligning responsibility with utilization.
* **Observability & controls:** Allocators monitor `allocatorDebt()`, `reservedZeroX`, pocket balances/allowances, UCE reserve levels, and the **dynamic redemption fee**. Per-asset **pause** and **haircut** policies allow governance risk responses (e.g., depegs) without compromising determinism.

***

### 2) Governance Onboarding & Lifecycle Management

Allocator privileges and limits are **ADMIN-gated** and executed through the governance layer (timelocked, multi-sig/EIP-712). Onboarding and updates are deterministic and auditable.

**Onboarding (end-to-end)**

* **Grant role & limits:** Governance executes a **Set Allocator** action with:\
  `allowed=true`, credit line `{ ceiling, dailyCap }`, and `borrowFeeBps`.\
  UCE records state, emits events, and deterministically **generates/maps a referral code** from `(allocator, ceiling, dailyCap, borrowFeeBps)`.
* **Pocket assignment:** Governance associates **per-asset pockets** (can batch or single-asset rotate later). Migration moves **up to min(balance, allowance)** from old to new pocket - never beyond safe bounds.
* **Operational readiness:**
  1. Fund pockets and set allowances to [UCE](../uce/);
  2. Pre-mint 0x inventory within dailyCap/ceiling;
  3. Distribute [referral code](../uce/concepts/allocators/referral-vs-non-referral.md);
  4. Set monitoring for reserves, allowances, fees, and index moves.

**Ongoing operations**

* **Credit mint (issuance):** Increases `reservedZeroX`, adds `baseDebt` = `amount × 1e18 / debtIndex`, updates daily counters, and enforces **ceiling/dailyCap**.
* **Repay (retire debt):** Send underlying to UCE; protocol skims **borrow fee** (bps) to treasury; principal is normalized (decimals only) to 0x-equivalent and caps at current [**effective debt**](../uce/concepts/allocators/debt-accounting-with-debt-index.md); `baseDebt` and global totals reduce; list order rebalances down.
* **Routing & settlement semantics:**
  * **Referred A→0x:** allocator’s **reserved 0x** is the sole source for user 0x (hard requirement).
  * **Protocol A→0x:** unreserved 0x → **pro-rata allocator draw** (nets debt) → mint remainder.
  * **0x→A:** allocator addresses cannot redeem 0x; user redemptions source underlying from **on-hand reserve, referral pocket, then global pocket -** all bounded by allowance/balance.
* **Policy changes & risk actions:** Governance may resize credit lines, toggle `allowed`, adjust `borrowFeeBps`, rotate referral codes, or rotate pockets; per-asset **pause**, **reserveBps**, **tin**, and **oracle** parameters can be tuned without changing allocator semantics.
* **Termination / throttle:** Disabling `allowed`, shrinking ceilings (never below current effective debt), or tightening allowances/pocket funding will gracefully throttle allocator activity; settlement invariants (bounded pulls, preview parity, index-consistent debt math) remain intact.

**Bottom line:** Allocators monetize distribution and yield while providing the system’s firs&#x74;**-**&#x6C;ine liquidity. Governance defines bounds; the engine enforces **deterministic settlement**, **index-consistent debt accounting**, and **referral-exact routing**, ensuring fairness and solvency across all flows.
