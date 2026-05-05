# Role in Money Market & Yield Distribution

Liquidity Providers play a dual role in ORBT’s ecosystem:

1. they **stabilize liquidity** and&#x20;
2. **fuel yield generation**.

***

#### **Capital Foundation**

LPs supply core stable assets (USDC, USDT, DAI, etc.) into ORBTMM.\
These deposits:

* Form the liquidity pool from which redemptions and intent settlements draw capital.
* Enable continuous minting and burning of 0xAssets by providing collateral backing.
* Support Allocators and Facilitators in executing real-time settlements.

Each asset deposited is divided into:

* **Reserve Layer (e.g., 25%)** → Held in UCE for instant redemptions.
* **Yield Layer (e.g., 75%)** → Supplied into Aave v3 to earn base yield through aTokens.

#### **Yield Distribution Pathways**

The yield earned from these deployments flows back through multiple channels:

* **To LPs** — as a baseline supply yield proportional to their share.
* **To the Treasury** — as protocol income used for ORBT buybacks and stability reserves.
* **To ORBT Stakers** — through governance-defined reward mechanisms (e.g., 0xUSD savings rate).
* **To Facilitators/Allocators** — as incentive spreads for efficient liquidity routing.

{% tabs %}
{% tab title="Example" %}
Alice deposits 1,000,000 USDC into ORBTMM.

* $250K is kept on-hand for redemptions.
* $750K goes to Aave, earning 5% APY → $37,500/year.
* Governance directs 70% of the yield ($26,250) to LPs, 20% ($7,500) to treasury, and 10% ($3,750) to ORBT stakers.\
  Alice’s net return ≈ 2.6% real yield — without emissions or inflation.
{% endtab %}
{% endtabs %}

#### **3. Real-Yield Sustainability**

Unlike inflationary farming models, ORBT’s yield is **derived from genuine market usage**:

* Lending interest from money markets.
* Facilitator performance fees.
* Borrowing spreads.
* Intent execution commissions.

This ensures that liquidity remains attractive even as emissions taper off — maintaining long-term equilibrium between supply, demand, and protocol profitability.
