# Parts to deal with

### Credit line & inventory

* Minting: allocatorCreditMint(allocator, amount) mints 0xAssets into UCE, credits the allocator’s reservedOx, and increases their effective debt (tracked as baseDebt × debtIndex).
* Debt accounting:
  * baseDebt → allocator’s nominal debt;
  * debtIndex → global scaler used for proportional adjustments (e.g., tail-loss socialization);
  * allocatorDebt(a) and totalAllocatorDebt() expose effective values.
* Repayment: allocatorRepay(asset, assets) pulls underlying from the allocator, skims borrow fee to treasury, converts the rest to 0x-equivalent, and reduces debt (capped to what’s outstanding).

### Pockets & routing

* Global pocket per asset + optional allocator-specific pocket per asset.
* Users can include a referral code; UCE maps it to the allocator and:
  * Sends the non-reserved portion of inbound stables to that allocator’s pocket.
  * On 0xAsset outflows, consumes the allocator’s reservedOx first if they’re the referrer.
* Reserve buffer: UCE retains reserveBps on-hand per asset (e.g., 25%) for low-latency settlements; the rest sits in pockets and can be deployed to Aave.

#### Redemption rate & tail-loss sink (why inventory matters) <a href="#id-3.3-redemption-rate-and-tail-loss-sink-why-inventory-matters" id="id-3.3-redemption-rate-and-tail-loss-sink-why-inventory-matters"></a>

* If users redeem 0xAssets for stables and the system needs to mint to meet demand, a time-varying redemption surcharge rises to price scarcity.
* If net shortfall persists, debt scaling lowers debtIndex, writing down indebted allocators pro rata. Keeping inventory pre-provisioned reduces the chance your flow hits scarcity conditions.

<br>
