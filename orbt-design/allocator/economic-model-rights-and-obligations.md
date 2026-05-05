# Economic model (rights & obligations)

### Rights

* [Credit line](../uce/concepts/allocators/0xasset-credit-minting.md) of 0xAssets- Governance sets a per-allocator ceiling (max effective debt) and a dailyCap (issuance velocity).
* Borrowing cost- The protocol targets a fix annual borrowing rate policy for allocators. In contracts today, this is implemented as a borrow fee in basis points charged at repay (i.e., on underlying delivered back when retiring debt). Governance can tune `borrowFeeBps` per allocator so the realized cost aligns with policy.
* Flow attribution- When a user provides a [referral code](../uce/concepts/allocators/referral-vs-non-referral.md) mapped on-chain to an allocator, the engine:
  * Routes inbound stables to that allocator’s pocket, and
  * Consumes that allocator’s reserved 0x inventory first when paying out 0xAssets. This lets allocators monetize their distribution and keep their inventory turning over.

### Obligations

* Custody & readiness- Allocators must maintain secure custody of pocket funds and keep adequate allowances so UCE can pull instantly for settlements.
* Provision inventory- Keep `reservedZeroX` topped up so users referred to you receive excellent fill quality without triggering the system’s scarcity surcharge path.
* [Repay debt](../uce/concepts/allocators/allocator-repayment.md)- Use `allocatorRepay(asset, assets)` to return underlyings and retire credit; treasury fees are skimmed at repay according to `borrowFeeBps`.
* Operate within limits- Respect `ceiling` and `dailyCap`. The contracts enforce both; mints revert if limits would be exceeded.
* Bear tail risk- In extreme scarcity where the engine must mint to meet redemptions, [debt scaling](../0xusd/debt-scaling-proportional-socialization-onto-issuers.md) socially amortises shortfall across indebted allocators in proportion to open credit. This aligns issuance with responsibility.&#x20;

:information\_source: _Note on referral & debt_: referral-coded swaps do not auto-repay allocator debt. They consume the allocator’s reserved 0x inventory first and route underlying flow to their pocket - indirectly improving their position. Actual debt reduction happens when the allocator calls the repay function with underlyings.
