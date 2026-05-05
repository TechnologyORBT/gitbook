# How Aave Works

We operate in **Aave v3** (conceptually; chain-specific params vary):

* **Supply:** deposit USDC → receive **aUSDC**; balance grows as interest accrues.
* **Borrow:** a borrower takes USDC and receives **variable** or **stable** debt (stable often disabled for some stables).
* **Interest:** utilization-based curves with a kink; supply APR ≈ borrow APR × (1 − reserve factor) × utilization.
* **Risk settings:** per-asset **LTV**, **liquidation threshold**, **liquidation bonus**, **caps**; optional **eMode** (higher LTV for correlated assets).
* **Credit delegation:** the supplier authorizes a borrower to draw debt **against the supplier’s collateral** by calling approveDelegation() on Aave’s debt token. The borrower can then borrow() without posting its own collateral. If the borrower fails to repay, **the delegator’s collateral is at risk**.

Our initial scope is simple: pockets **supply only** (no native borrowing). We enable **credit delegation** to whitelisted settlement actors for instant pulls, with strict caps so HF never approaches liquidation.
