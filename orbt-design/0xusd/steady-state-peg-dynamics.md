# Steady-State Peg Dynamics

* **Narrow band:** In liquid conditions, arbitrage via [UCE](../uce/) quotes keeps the external price of 0xUSD near $1 net of rounding. The [reserve buffer](../pocket/reserve-buffer-and-rebalancing.md) handles routine outflows; pockets handle bursts; [Allocators ](../uce/concepts/allocators/)expand/contract supply as spreads dictate.
* **Premium episodes:** Allocators mint and sell; users can [mint via asset](../../user-flow-and-guides/how-to-buy-0xassets.md) → 0xUSD with no redemption-rate impact; supply rises and premium compresses.
* **Discount episodes:** Users redeem 0xUSD → stablecoins; UCE burns 0xUSD, updates the [redemption rate](redemption-rate-and-shortfall-socialization.md), and may [scale allocator debts](../uce/concepts/backing-invariants.md#id-9-allocator-credit-debt-and-limits); supply falls and price lifts.
