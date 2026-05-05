# Debt Scaling (proportional socialization onto issuers)

* On redemptions, UCE burns the inbound 0xUSD and pays out the underlying at the oracle quote.
* UCE then adjusts the global debt index so that indebted Allocators absorb systemwide tail losses pro-rata to their effective debt:\
  \
  $$newIndex = oldIndex * (EffTotal - NetShortfall) / EffTotal$$\
  \
  &#xNAN;_&#x77;here EffTotal is the sum of effective allocator debts_ and\
  &#xNAN;_&#x4E;etShortfall is a redemption-rate-adjusted amount derived from the 0xUSD redeemed._\
  &#xNAN;_&#x49;f NetShortfall ≥ EffTotal, the epoch is wiped and debts are reset (with index re-initialized)._
* `User experience`: the redeemer receives the quoted underlying; the redemption rate does not haircut the user’s payout. The cost of stress is internalized by issuers via the index scaling.

### No Interest Rate on Issuance (underlying → 0xUSD)

* When users bring in stablecoins for 0xUSD and on-hand + reserved inventories are insufficient, UCE mints the shortfall 0xUSD to the receiver directly. No redemption rate or debt scaling is applied on this issuance path.
