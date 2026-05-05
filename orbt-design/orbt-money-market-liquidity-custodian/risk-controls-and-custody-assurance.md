# Risk Controls & Custody Assurance

**ORBTMM** employs multiple defense layers to ensure that liquidity across the ecosystem remains **secure, predictable, and fully auditable**.

The framework is designed to remain **money market agnostic**, enabling integration with **Aave v3** or any **vetted and trusted** [**money market**](../pocket/what-is-a-money-market/) that meets ORBT’s risk and liquidity standards.

#### **Custody Isolation**

* Each [**Pocket**](../pocket/) operates in isolation, ensuring that any failure or compromise within one does not affect others.
* Capital held within each Pocket is secured through **hardened multisignature** or **institutional-grade MPC custody solutions**.
* This structure provides operational resilience and fault isolation across strategies.

#### **Money Market Integration Safeguards**

ORBTMM can deploy capital into multiple **trusted money markets**, such as **Aave v3** or other **audited and battle-tested protocols** that meet internal liquidity and safety criteria.

Integration follows strict security and risk parameters:

* Only markets with **deep liquidity** and **stable asset pairs** are approved.
* Risk parameters such as **Loan-to-Value (LTV)**, **borrow caps**, and **liquidation thresholds** are configured conservatively, maintaining a **target Health Factor (HF) ≥ 2.2**.
* **Delegation caps** (for example, 15% global and 3% per delegate) limit exposure and prevent over-utilization.
* Protocol-level integration reviews and audits are mandatory before onboarding new markets.

This modular approach ensures ORBTMM remains flexible, allowing future integration with other vetted money markets while maintaining consistent safety standards.

#### **Real-Time Monitoring**

* Automated alerts for:
  * Low reserves or allowances
  * High Aave utilization (> 90%)
  * Excess delegated usage (> 50% cap)
  * Health Factor approaching risk thresholds
* Governance [timelocks](../allocator/governance-and-policy-layer-setup.md#id-5.3-timelock) and circuit breakers provide emergency recovery options.

#### **Governance Control & Insurance**

* [Governance ](../allocator/governance-and-policy-layer-setup.md)retains full authority to **pause credit delegation**, **limit exposure**, or **halt capital deployment** during periods of high volatility or abnormal behavior.
* An **Emergency Council multisig** can execute immediate protective actions when necessary.
* Dedicated **Insurance Modules** and **Safety Vaults** are maintained to absorb losses in extreme cases, such as a protocol exploit or bridge-related impact.
