# Redemption Rate and Shortfall Socialization

Fix applied: The redemption rate is applicable when users swap from 0xUSD to underlying. It is not applied when users go from underlying to 0xUSD.

#### Dynamic Redemption Rate (update on redemption)

* Triggered only on 0xUSD → underlying conversions.
* The engine updates baseRedemptionRate proportionally to the redeemed fraction of total 0xUSD supply and decays it over time (\~0.5% hourly, capped, e.g., 5%).

Economically, it is a stress meter: heavy redemptions raise the rate; quiet periods let it decay.
