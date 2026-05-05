# System Actors and Inventory Context

* **Users**: Swap between 0xUSD and supported stablecoins; optionally interact with [s0xUSD ](../../user-flow-and-guides/how-to-buy-s0xasset.md)(ERC-4626 wrapper on 0xUSD).
* [**Allocators**](../uce/concepts/allocators/): Permissioned market-makers with a line of credit (ceiling, dailyCap) to mint 0xUSD, warehouse it as `reservedZeroX`, and manage per-asset pockets that deploy supported stablecoins into external strategies (Aave, Maker DSR/sDAI, T-bill wrappers, Curve pools, etc.).
* [**UCE** ](../uce/)**(the engine)**: The settlement hub. Holds on-hand 0xUSD and on-hand underlyings (reserve buffer) and routes to pockets via allowances. Updates the dynamic redemption rate and performs debt scaling on 0xUSD → underlying conversions.
* **Treasury**: Receives allocator borrow fees (assessed on allocator repay).
