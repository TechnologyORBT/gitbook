# Facilitator Incentive & Risk Structure

Facilitators are **profit-driven participants** who earn revenue from the strategies they operate. ORBT’s model is designed to align facilitator profitability with protocol sustainability through a structured combination of fees, revenue sharing, and accountability mechanisms.

#### **Flat Protocol Fee**

Facilitators pay a **regular fee** to the ORBT protocol for access rights.\
This can be structured as either:

* A **fixed fee** (e.g., a recurring 0xUSD payment), or
* A **percentage-based fee** on assets under management.

This model ensures the protocol earns baseline revenue even during low-activity periods and acts as a “platform franchise cost.”

#### **Profit Sharing**

Facilitators share a portion of their **net profits** with the ORBT Treasury.\
For instance, if a facilitator earns **$100,000** in trading profits under a **20% profit-share**, then **$20,000** goes to ORBT and **$80,000** remains with the facilitator.\
Splits are **configurable per facilitator or strategy type**, balancing fairness with incentive alignment.

#### **Operational Costs**

Facilitators cover their own **infrastructure and gas expenses**.\
However, ORBT governance may allocate **gas stipends or reimbursements** for operations that directly benefit protocol stability — such as peg arbitrage or essential liquidity rebalancing — ensuring facilitators are not penalized for performing low-margin but critical functions.

#### **Risk and Accountability**

Facilitators operate under strict governance-defined risk controls:

* **Bucket enforcement:** Minting and deployment caps are enforced at the smart-contract level.
* **Performance monitoring:** On-chain metrics track utilization, yield, and compliance.
* **Bonding (optional):** Governance may require facilitators to stake ORBT tokens or collateral as economic security.
* **Revocation:** Governance can suspend or revoke facilitator privileges for underperformance, violations, or security issues.
* **Slashing (governance-defined):** Misconduct resulting in protocol losses may lead to partial or full loss of bonded collateral.

***

### **Transparency and Accountability**

All facilitator activities are **fully transparent and on-chain**. ORBT governance — and independent auditors — can monitor:

* **Bucket usage:** Current mint levels versus total capacity.
* **Performance metrics:** Pocket profitability, execution success rates, and yield generation.
* **Compliance:** Adherence to risk parameters and operational guidelines.

Governance may require **periodic reporting or audits**.\
If a facilitator underperforms or violates rules, governance can **reduce their bucket capacity**, **freeze their mint permissions**, or **remove them entirely** using functions such as `removeFacilitator()`.

For example, if a facilitator is found engaging in high-risk trades outside their mandate, the DAO can immediately revoke privileges through a governance vote — ensuring rapid mitigation of systemic risk.

### **Example: Institutional Facilitator (AlphaCorp)**

Consider **AlphaCorp**, an institutional facilitator approved to operate pre-funded settlement strategies.\
AlphaCorp partners with businesses needing large on-chain forex swaps. It mints **50 million 0xUSD** (with collateral backing or governance-approved credit), and deploys the funds into yield-generating Pockets until the settlement date.

When settlement occurs, AlphaCorp uses ORBT Pockets to execute swaps, leveraging **ORBTMM** liquidity for instant settlement.\
Counterparties receive guaranteed liquidity, AlphaCorp earns transaction fees, and the ORBT protocol captures a share — for example, **0.1% of volume plus 10% of net profit**.

This model contributes to ORBT’s revenue from **pre-funded institutional intent settlements**, a key driver behind early growth (e.g., $10.5B in processed settlement flows).

***

### **Facilitators as Peg & Liquidity Keepers**

Some facilitators are specifically tasked with **peg stabilization and liquidity balancing**.\
For example, a facilitator might operate a **PSM arbitrage bot**: when **0xUSD** deviates from $1, the facilitator arbitrages through the PSM — minting or burning 0xUSD to restore parity.\
This activity maintains the system’s stability and generates modest profit, part of which flows to the protocol treasury.

The DAO may offer **incentive boosts or micro-rewards** for such operations, recognizing their role in maintaining ecosystem health.\
This structure parallels **MakerDAO’s keeper network** and other protocols’ **market operations teams**.

***

Facilitators can voluntarily **exit the system** by unwinding their positions — burning any remaining 0xAssets, closing all active Pockets, and settling outstanding obligations.\
Upon completion, governance can remove them cleanly from the protocol’s registry.

If a facilitator **misbehaves** — for example, by failing to pay fees or violating strategy parameters — the DAO can take immediate action:

* **Freeze bucket permissions** (preventing new mints or deployments).
* **Invoke emergency removal** through governance functions such as `removeFacilitator()`.
* **Slash collateral or stake**, if required by governance policy.

This ensures a swift, transparent, and on-chain enforcement mechanism to protect the protocol and its users.
