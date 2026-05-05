# Risk Controls & Custody Assurance

ORBTMM employs multiple defense layers to ensure that **liquidity is secure, predictable, and auditable.**

#### **Custody Isolation**

* Each Pocket is independent: a failure in one does not affect others.
* Pockets use **hardened multisigs** or institutional MPC custody for capital safety.

#### **Aave Integration Safeguards**

* ORBTMM only uses **Aave v3 markets** with deep liquidity and stable asset pairs.
* Parameters such as LTV, borrow caps, and liquidation thresholds are set conservatively (HF target ≥ 2.2).
* Delegation caps (e.g., 15% global, 3% per delegate) prevent over-utilization.

#### **Real-Time Monitoring**

* Automated alerts for:
  * Low reserves or allowances
  * High Aave utilization (> 90%)
  * Excess delegated usage (> 50% cap)
  * Health Factor approaching risk thresholds
* Governance timelocks and circuit breakers provide emergency recovery options.

#### **Governance Control & Insurance**

* Credit delegation or high-impact operations can be paused instantly via governance or the **Emergency Council multisig**.
* Insurance modules and **Safety Vaults** stand ready to absorb losses if a counterparty defaults or a bridge event impacts liquidity.
