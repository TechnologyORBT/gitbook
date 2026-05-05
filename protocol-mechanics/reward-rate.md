---
hidden: true
---

# Reward Rate

#### What it is

The **ORBT Rewards Rate (ORR)** distributes **ORBT token incentives** to accounts that allocate [**0xAssets** ](../orbt-design/0xassets.md)to the Rewards Module.

Use 0xAssets to access ORBT Staking Rewards and get the Alpha from the ORBT ecosystem and a direct chunk of revenue dedicated to the ORBT token stakers. No minimum 0xAsset supply amount is required. This feature is non-custodial.

When you supply 0xAssets to the ORBT Token Rewards module of the ORBT, you get ORBT Token Rewards over time in the form of ORBT governance tokens.

With ORBT, you always remain in control of your funds.

{% columns %}
{% column width="50%" %}
<h2 align="center"><sub>ORBT Rewards Rate</sub></h2>

<h2 align="center"><sup><sub><em>Target:</em></sub></sup>   <mark style="color:$primary;">7%</mark></h2>
{% endcolumn %}

{% column width="50%" %}
<h2 align="center"><sub>ORBT Token Rewards TVL</sub></h2>


{% endcolumn %}
{% endcolumns %}

## How ORBT Staking Rewards Work

Eligible users may supply any supported **0xAsset** to the **ORBT Staking Rewards** module to begin accruing rewards automatically.

**Reward rates are variable** and differ by rewarded token. They fluctuate based on:

* The **emission schedule** for the rewarded token set by **ORBT governance**
* The **market price** of the rewarded token
* The user’s **share of total 0xAssets** supplied to the Rewards module

**Control & execution:**\
The **ORBT front-end** does not control reward issuance, parameters, or distribution. These functions are executed by protocol smart contracts under governance.

#### How it’s set (Governance policy)

* **Target incentives:** Governance sets a target **Rewards APR** (displayed as a percent) by specifying an **ORBT emissions schedule**.
* **Adaptive calibration:** Emissions can be re-weighted across pools and epochs to keep the displayed APR near target given TVL drift and price changes.

#### Where rewards come from

* **Token emissions:** ORBT distributed per the governance-approved schedule.

#### Current set rate

* **ORR target:** **7% APR** (estimate, based on current emissions, TVL, and price oracles).
* Displays are **informational** and adjust with TVL/emissions; on-chain accounting is the source of truth for accrued ORBT.

### Policy & Risk Notes

* **Continuity:** ORR is variable and may be revised by governance with notice and [timelocks](../orbt-design/allocator/governance-and-policy-layer-setup.md#id-5.3-timelock).
* **Sustainability:** ORR emissions are capped by the emissions budget.
* **Volatility & liquidity:** Displayed APRs for ORR depend on token price and TVL; sudden inflows can dilute per-account rates.
* [**Security**](../security-and-safety/security.md)**:** Smart-contract risk, oracle/display risk, and policy risk apply; audits and emergency controls are documented in the resources section
