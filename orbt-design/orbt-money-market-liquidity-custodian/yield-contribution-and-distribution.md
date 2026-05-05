# Yield Contribution & Distribution

ORBTMM is designed to ensure that every dollar of capital earns measurable yield while staying redeemable.

#### **Aave-Only Strategy**

* Non-reserved funds in Pockets are supplied to Aave → receive aTokens.
* Interest accrues continuously as aTokens appreciate via index scaling.
* A small **hot buffer (1–3%)** is retained on-Pocket for micro-settlements, avoiding constant withdraw/re-supply cycles.

#### **Rebalancing Logic**

* **UCE Reserves:** Parameterized by `reserveBps` (e.g., 25%).
* If reserves < threshold → pull from Pocket (or withdraw from Aave).
* If reserves > ceiling → push excess to Pocket → supply to Aave.
* The system maintains equilibrium between immediate liquidity and optimal yield.

#### **Revenue Flow**

* Yield generated in Pockets contributes to:
  * **Protocol revenue** (shared with treasury & stakers).
  * **Allocator carry**, incentivizing proactive liquidity management.
  * **Stable savings yield** for 0xUSD holders when distributed via governance.

{% tabs %}
{% tab title="Example" %}
Pocket deposits $5M USDC → earns 5% APY on Aave.\
$250K/year yield flows into ORBT’s treasury.

Governance allocates 60% of that to stakers, 20% to treasury, and 20% to allocator performance fees.\
This creates a circular value loop — <mark style="background-color:yellow;">yield → revenue → buyback & burn → token value accrual.</mark>
{% endtab %}
{% endtabs %}
