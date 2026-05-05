# Interaction with Facilitators & Governance

Liquidity Providers are indirectly tied to Facilitators and Governance — their yield, liquidity health, and long-term returns depend on how these layers operate and evolve.

#### **With Facilitators**

* Facilitators utilize liquidity for real-world and DeFi settlements (e.g., arbitrage, market-making, RWA flow).
* LP capital backs these operations indirectly through **credit delegation** from Pockets.
* Successful facilitator operations increase utilization → yield → LP returns.
* Mismanagement risk is mitigated by delegation caps, HF guards, and governance-imposed limits.

{% tabs %}
{% tab title="Example" %}
A facilitator managing telecom settlements uses ORBTMM liquidity to process $10M in 0xUSD cross-chain transactions.

The resulting transaction fees (0.2%) feed back into protocol yield.\
LPs benefit from both Aave interest and a share of facilitator revenue, distributed quarterly by governance.
{% endtab %}
{% endtabs %}

#### **With Governance**

Governance determines:

* Yield distribution ratios between LPs, treasury, and stakers.
* ReserveBps thresholds (liquidity buffer ratios).
* Approved money market integrations.
* Fee schedules for minting, redemption, and facilitator usage.

LPs can participate in governance directly by **staking ORBT** or indirectly by delegating votes.\
Governance votes ensure transparency in how yield is split, how risk is managed, and how the treasury reinvests surplus.

{% tabs %}
{% tab title="Example" %}
A DAO proposal might increase LP yield share from 60% → 70%, reducing treasury take to incentivize more deposits during expansion phases.\
LPs who stake ORBT can vote in favor or delegate to trusted governance agents.
{% endtab %}
{% endtabs %}

#### **Transparency & Data Access**

LPs have full visibility into:

* Total liquidity supplied and utilization via **subgraph endpoints**.
* Pocket-level allocations (reserve vs. Aave vs. delegated credit).
* Real-time APYs, redemption queue depth, and reserve health metrics.
* Governance changes affecting yield or liquidity parameters.

ORBT ensures **on-chain transparency** and **auditable reporting**, allowing LPs to make informed decisions and manage exposure confidently.
