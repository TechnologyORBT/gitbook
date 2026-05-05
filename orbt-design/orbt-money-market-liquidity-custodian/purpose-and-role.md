# Purpose & Role

ORBTMM’s primary purpose is to **balance liquidity availability with capital productivity**.

It plays four major roles in the ORBT architecture:

1. **Liquidity Custodian**\
   ORBTMM holds and manages all active and idle liquidity across supported stablecoins (e.g., USDC, USDT, DAI).\
   Each asset has a **global pocket** and may include **allocator-specific pockets**, enabling distributed yet coordinated liquidity.
2. **Settlement Engine**\
   ORBTMM provides **instant redemptions** through its on-hand reserves and credit-delegated capacity from Pockets.\
   When a user redeems 0xUSD, the UCE first uses its reserve buffer; if reserves dip, ORBTMM pulls directly from Pockets via ERC-20 allowances.
3. **Yield Generator**\
   Idle capital within Pockets is automatically deployed into **Aave v3** or other vetted money markets.\
   These deposits earn a conservative **base yield** while remaining available for withdrawal at any time.
4. **Risk-Balanced Allocator Interface**\
   ORBTMM enables **Allocators** to manage liquidity responsibly — pre-funding pockets, maintaining reserves, and earning yield spreads.\
   This turns allocators into active liquidity partners who absorb operational risk and generate market depth.

{% tabs %}
{% tab title="Example" %}
When an allocator deposits $10M USDC into ORBT, 25% ($2.5M) remains in the UCE’s on-hand buffer, while 75% ($7.5M) moves to the allocator’s Pocket.\
That $7.5M is supplied to Aave, earning \~5% APY. When a redemption or intent arises, ORBTMM can instantly pull liquidity from that Pocket or use delegated borrowing to fulfill the request — maintaining both speed and solvency.
{% endtab %}
{% endtabs %}
