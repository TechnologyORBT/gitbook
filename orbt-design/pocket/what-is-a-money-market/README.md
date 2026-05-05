# What is a Money market

A **money market** (Aave/Compound-style) is a pooled lending protocol:

* **Suppliers** deposit assets into a shared pool and receive **interest-bearing tokens** (e.g., Aave’s **aTokens**).
* **Borrowers** draw from the pool by posting collateral; interest paid by borrowers funds supplier yield.
* **Rates** float with **utilization**: as more of the pool is borrowed, borrow rates rise (often with a “kink” step) and supply rates follow.
* **Risk controls:** per-asset collateral factors (LTV), liquidation thresholds, reserve factors, supply/borrow caps, isolation lists, and (in Aave v3) features like **eMode** for correlated assets.
* **Composability:** aTokens are standard ERC‑20s accruing interest, usable in other protocols.

Important invariants:

* Liquidity is pooled: withdrawals succeed if pool liquidity is available; otherwise they may be rate-limited or partially filled until liquidity returns.
* Interest accrues continuously via indices maintained by the protocol; aTokens increase in value by balance growth (rebasing) or index scaling.
