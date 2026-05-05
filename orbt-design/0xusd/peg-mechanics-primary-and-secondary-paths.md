# Peg Mechanics: Primary and Secondary Paths

### **A) 0xUSD Premium (> $1 externally)**

Arb supply expands.

1. Allocators mint 0xUSD via [credit lines](../allocator/parts-to-deal-with.md#credit-line-and-inventory) (no redemption-rate effect at issuance) to build inventory and sell externally above $1, then later repay with supported stablecoins (paying `borrowFeeBps`). `Profit = market premium − financing costs`.
2. Users swap stablecoins → 0xUSD at oracle quotes. If on-hand + reserved inventory is insufficient, UCE mints 0xUSD to the receiver to settle the trade. No redemption rate is applied on this path (issuance path). → Both flows increase circulating 0xUSD where it trades rich, pushing price back toward $1.

### **B) 0xUSD Discount (< $1 externally)**

Arb demand retires supply.

1. Users buy 0xUSD cheap externally and swap 0xUSD → stablecoins at UCE near $1 (oracle quote). UCE burns inbound 0xUSD and pays out supported stablecoins from on-hand reserves and pockets. This path updates the [dynamic redemption rate](../uce/concepts/dynamic-redemption-fees.md) and can trigger allocator [debt scaling](debt-scaling-proportional-socialization-onto-issuers.md).
2. Allocators buy back 0xUSD at a discount, reduce `reservedZeroX`, and/or repay their debt with stablecoins. → Circulating 0xUSD shrinks; price lifts toward $1.

**Result**: Convertibility bands (plus rounding) create two-sided arbitrage that pins 0xUSD near $1 in normal liquidity.<br>
