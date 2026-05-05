# Allocator Incentives and Risk

* **Carry Model**: Allocators earn external spread when selling 0xUSD rich and can farm stablecoin yields in pockets. They pay borrow fees (`borrowFeeBps`) when repaying. Credit ceilings and daily caps bound exposure and issuance velocity.
* **Inventory Obligation:** Serving referred flow requires maintaining `reservedZeroX` and pocket liquidity; failing to pre-provision pushes users into engine minting on the issuance path (still seamless for users) but leaves allocators exposed to future redemption-driven debt scaling.
* **Strategy Choice:** Pockets should favor low-volatility, short-exit-time venues; allowances and buffers must be sized so [UCE](../uce/) pulls settle promptly.
