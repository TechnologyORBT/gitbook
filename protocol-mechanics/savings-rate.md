# Savings Rate

**ORBT Savings Rate (OSR)** – A variable, governance-controlled yield on [**0xUSD**](../orbt-design/0xusd/) that accrues continuously to depositors in the Savings module. No minimum supply amount is required; withdraw anytime.&#x20;

With ORBT, you always remain in control of your savings, as this feature is non-custodial.

### What it is

The **ORBT Savings Rate (OSR)** is a variable, governance-controlled yield on **0xUSD** deposited into the Savings Module. Deposits mint [**s0xUSD**](../user-flow-and-guides/how-to-buy-s0xasset.md), a transferable receipt token whose **exchange rate vs. 0xUSD rises over time**, reflecting continuously accrued savings.

{% columns %}
{% column width="50%" %}
<h2 align="center"><sub>ORBT Savings Rate</sub></h2>

<table><thead><tr><th width="159">Currency</th><th>Target Savings (rebased)</th></tr></thead><tbody><tr><td>BTC</td><td><mark style="color:$info;">4%</mark></td></tr><tr><td>ETH</td><td><mark style="color:$info;">5%</mark></td></tr><tr><td>SOL</td><td><mark style="color:$info;">7%</mark></td></tr><tr><td>Stables</td><td><mark style="color:$info;">10%</mark></td></tr></tbody></table>
{% endcolumn %}

{% column width="50%" %}
<h2 align="center"><sub>Savings TVL</sub></h2>


{% endcolumn %}
{% endcolumns %}

### How it works: s**0xAssets** - your savings token

When you supply **0xAssets** to the **ORBT Savings Rate (OSR) module** of the decentralized ORBT Protocol, you mint **s0xAssets**. These tokens are the on-chain record of your position and the value that accrues to it over time.

The protocol **continuously credits** additional 0xAssets to the module **in line with the OSR** (accruing per second). As the pool grows, the **exchange rate of s0xAssets ↔ 0xAssets increases**, so simply holding s0xAssets reflects your share of the expanding pool.

When you choose to [redeem](../user-flow-and-guides/how-to-sell-s0xassets.md), you can **burn s0xAssets at any time** to withdraw **0xAssets equal to your original deposit plus** [**accrued savings**](../user-flow-and-guides/how-to-claim-yield.md) (subject to network costs and any applicable protocol fees). s0xAssets are transferrable and composable, so they can be used across DeFi while your savings continue to accrue.

### How it’s set (Governance policy)

* **Target rate:** Governance sets a target **annualized** OSR as a policy lever for liquidity depth, peg stability, and ecosystem growth.
* **Revenue discipline:** Governance maintains the OSR **subject to a budget constraint** after allocating **70% of net protocol revenue** to **continuous ORBT** [**buyback & burn**](orbt-revenue-and-yield-model/immutable-buyback-engine.md#burn-and-supply-reduction). In practice:
  * The **residual** revenue and designated reserves must **sustain** OSR outflows; if not, governance reduces the OSR target.
* **Change management:** Adjustments are executed via [timelocked proposals](../orbt-design/allocator/governance-and-policy-layer-setup.md#id-5.3-timelock) with published rationale and forward guidance.

### Where yield comes from

OSR is funded by protocol cash flows, including (non-exhaustive):

* **Set Interest from the** [**Allocators**](../orbt-design/uce/concepts/allocators/)
* **Settlement fees** from the on-chain Clearing House (net of operator distributions).
* [**Money Market**](../orbt-design/pocket/aave-only-pocket-strategy/) interest margin, liquidation surplus, and fee income.

### Current set rate

* **OSR (Savings) target:** **7% APY** (annualized), accruing per-second.
* Displays show the **annualized equivalent** of the on-chain accumulator; realized returns track the accumulator, subject to policy changes.

### Policy & Risk Notes

* **Continuity:** OSR is variable and may be revised by governance with notice and timelocks.
* **Sustainability:** OSR is bounded by realized revenue **after** the 70% buyback-and-burn allocation
* **Composability:** s0xUSD remains transferrable and usable in external protocols while continuing to accrue OSR; Rewards balances accrue linearly and are claimable on-chain.
* **Security:** Smart-contract risk, oracle/display risk, and policy risk apply; audits and emergency controls are documented in the Security & Governance sections.
