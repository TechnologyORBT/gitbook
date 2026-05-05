# Custody & Segregation Models

Security is paramount for a protocol that handles unified liquidity across chains and large volumes of capital, including institutional funds.\
ORBT’s security model is **multilayered**, covering **smart contract safety**, **custody of assets**, **oracle reliability**, and **governance resilience**.

Given the “permissioned” nature of **Pockets** and **facilitator roles**, operational security and accountability are equally critical to prevent misuse.\
This section details how ORBT safeguards user funds and system integrity through its architecture, processes, and governance.

#### **Smart Contract Custody**

Assets within the ORBT system are held **on-chain** in smart contracts such as the **UCE vaults**, **UCE reserves**, and **Pocket contracts**.\
These contracts are **non-custodial**, meaning users always retain control and can withdraw assets as defined by protocol rules — no admin or facilitator can redirect these funds elsewhere.

This ensures **self-custody via code**, minimizing any scenario where assets are in limbo or could be arbitrarily moved.

#### **Multisig Controls**

Certain administrative functions use **multisignature wallets** for added control and decentralization. Examples include:

* **DAO Treasury:** managed via a multi-signature wallet (e.g., 4-of-7 Gnosis Safe).
* **Emergency Pause Keys:** held by a trusted security council (e.g., 3-of-5 multisig).
* **Bridge Custody:** temporary control during asset bridging (e.g., Ethereum ↔ Polygon).

ORBT will publicly disclose **multisig addresses** and **signers (by role)**. Over time, these responsibilities will transition from the core team to **community-elected signers**.

#### **Institutional Custody and EOAs**

Some institutional participants may use their own custody solutions (e.g., Fireblocks, MPC wallets, or Ledger-based multisigs).\
In such cases, these **EOAs** are **whitelisted** to receive and manage ORBT liquidity via **ORBTMM**.

To mitigate risks:

* These EOAs must be **multisig or MPC-based**,
* They may be **insured or bonded**,
* And **facilitator-specific limits** apply to exposure.

For most users, **native smart contract Pockets** remain the recommended and safer approach.

#### **Cross-Chain Bridge Security**

As a **chain-agnostic protocol**, ORBT relies on secure bridging frameworks.\
Preferred solutions include:

* **Canonical Bridges** from major providers (e.g., Circle’s CCTP for USDC).
* **Trust-minimized Bridges** such as **Wormhole**, **LayerZero**, or **Axelar**, evaluated based on risk.
* **Hybrid approaches** like **Chainlink CCIP** for cross-chain messaging.

When transferring 0xUSD or other assets cross-chain, ORBT limits risk exposure by:

* Capping bridge transaction volumes,
* Segregating bridge contracts by asset type,
* Maintaining isolation if a bridge fails.

Spark’s SLL, for instance, uses both SkyLink (Maker’s own bridge) and Circle CCTP for different assets. When ORBT sends assets cross-chain (like bridging 0xUSD), it will do so in a way that if a bridge fails, the damage is isolated.

Additionally, ORBT may **mint native 0xUSD** per chain — managed by facilitators — instead of bridging the same token repeatedly. This mirrors **MakerDAO’s Teleport mechanism**, improving safety and liquidity efficiency.

{% hint style="info" %}
If bridging is needed, ORBT will cap how much can be in transit or on a particular chain’s custody contract at once, to limit exposure.
{% endhint %}

#### **Native Minting per Chain**

Rather than relying solely on bridges, ORBT may **natively mint 0xUSD and other 0xAssets** on multiple chains.

Each chain’s supply is **managed by facilitators** through governance-approved quotas (similar to MakerDAO’s **teleport** model for DAI).\
This design avoids persistent bridge custody and allows **faster, safer liquidity movement** between ecosystems without direct asset transfers.

It also improves recovery: if one bridge fails, ORBT can rebalance supply using other chains’ liquidity, keeping parity through UCE arbitrage.

#### **Governance Attack Mitigation**

Because ORBT governance controls major parameters, **governance attacks** pose real risks.\
Mitigation measures include:

* **DAO time-locks** on proposals (delayed execution, e.g., 48 hours).
* **Emergency veto rights** early in launch phase (team or council-based).
* **Safety modules** for staking — malicious voters risk losing stake.
* **Broad token distribution** to make hostile accumulation expensive.

Transparent governance combined with wide liquidity listings prevents manipulation and centralization.

#### **Real-World Asset (RWA) Custody**

**Custody Structure**

ORBT’s dual-yield design may involve allocations into **real-world assets (RWA)** like short-term treasury bills, managed through **regulated custodians** or **special purpose vehicles (SPVs)**.

* These SPVs act as legally ring-fenced entities that hold underlying collateral (e.g., treasuries or commercial paper) on behalf of ORBT tokenholders.
* Partner custodians, such as licensed trust companies, manage these assets transparently under defined agreements.
* Each RWA allocation is tokenized, enabling on-chain tracking of ownership and yield streams, bridged through verified oracles.

This hybrid approach combines **DeFi transparency** with **TradFi reliability**.

**Risk Mitigation and Governance Oversight**

Governance ensures:

* Only pre-approved, **reputable asset managers** handle RWA custody.
* Strict **allocation limits** apply (e.g., a maximum of 25–50% of reserves invested in RWA).
* **Regular audits** and **proof-of-reserve attestations** confirm that off-chain assets match their on-chain representations.

DAO votes are required for any new RWA inclusion or changes to custodial relationships, ensuring decentralized oversight of otherwise off-chain assets.

**Insurance and Backstop Integration**

If RWA exposure leads to loss (e.g., default of a counterparty or government seizure), the **Insurance Fund** or **Safety Vaults** can intervene, absorbing partial or full losses depending on severity.

Additionally, ORBT governance can vote to **inject tokens from the staking reserve** or **trigger buybacks** to recapitalize shortfalls — creating a controlled recovery loop instead of ad-hoc crisis responses.
