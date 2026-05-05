# Steady-State Peg Dynamics

* **Narrow band:** In liquid conditions, arbitrage via UCE quotes keeps the external price of 0xUSD near $1 net of rounding. The reserve buffer handles routine outflows; pockets handle bursts; Allocators expand/contract supply as spreads dictate.
* **Premium episodes:** Allocators mint and sell; users can mint via underlying → 0xUSD with no redemption-rate impact; supply rises and premium compresses.
* **Discount episodes:** Users redeem 0xUSD → stablecoins; UCE burns 0xUSD, updates the redemption rate, and may scale allocator debts; supply falls and price lifts.
