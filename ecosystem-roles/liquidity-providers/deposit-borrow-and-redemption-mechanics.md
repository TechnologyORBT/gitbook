# Deposit, Borrow, and Redemption Mechanics

The ORBTMM allows LPs to interact directly with the protocol through a **simple but powerful flow**.

#### **Depositing Liquidity**

LPs deposit supported stablecoins or assets into ORBTMM:

```solidity
function deposit(uint256 amount, address onBehalfOf)
```

* On deposit, LPs receive an internal accounting of their pool share (or an LP token, depending on integration).
* Deposits become part of the **active reserve** or are supplied to **Aave via Pockets** for yield.

**Behind the scenes:**

* 25% of deposits remain in the UCE reserve.
* 75% are forwarded to the Pocket linked to that asset and supplied to Aave.
* The system tracks LP balances via subgraph queries or the SDK.

{% tabs %}
{% tab title="Example" %}
LP deposits 500,000 USDC.

* 125K remains in UCE (reserve).
* 375K is deposited to Aave, generating aUSDC yield.\
  LP starts accruing interest proportional to their share of the pool.
{% endtab %}
{% endtabs %}

#### **Borrowing Mechanism (Facilitator-Driven)**

While individual LPs don’t borrow directly, their deposits enable borrowing through **Facilitators or credit-delegated actors**.

* Facilitators can borrow from Aave against collateral supplied by Pockets.
* These loans are short-term, intent-based, and **automatically repaid** once settlements complete.
* LPs indirectly earn interest from these borrowings as Aave’s supply rate rises with utilization.

{% tabs %}
{% tab title="Example" %}
A Facilitator executes a $200K arbitrage intent.

* Borrows instantly via delegated credit from the Pocket’s aUSDC.
* Repays within minutes with a small fee.
* LPs indirectly benefit from this utilization because Aave’s utilization curve spikes, increasing their supply yield.
{% endtab %}
{% endtabs %}

#### **Redemption Flow**

When users redeem 0xUSD or other 0xAssets:

1. The UCE burns the stablecoin and initiates payout.
2. Funds are sourced first from on-hand reserves.
3. If reserves fall short, ORBTMM pulls liquidity directly from the Pocket (via `safeTransferFrom`).
4. LP balances remain intact — the redemption is handled at protocol level.

Withdrawals follow a similar flow:

* LP requests withdrawal → UCE checks liquidity → pulls from Pocket or triggers Aave withdrawal.
* Withdrawals are gas-efficient and automated through the protocol router.

{% tabs %}
{% tab title="Example" %}
A redemption of $600K occurs:

* $250K comes from reserve.
* $350K is pulled from Pocket (instant transfer).
* LPs’ overall share remains the same, with the pool automatically rebalanced by the ORBTMM reserve logic.
{% endtab %}
{% endtabs %}

#### **Rebalancing and Safety Margins**

ORBTMM maintains dynamic equilibrium between reserve liquidity and yield deployment.

* **Reserve watermark:** Triggers replenishment from Pockets when reserves < threshold.
* **Ceiling level:** Pushes excess idle funds back to Aave when reserves > ceiling.
* **Rate limiters:** Prevent flash drains by throttling large simultaneous withdrawals.

This model ensures **low latency**, **predictable liquidity**, and **continuous yield accrual**.
