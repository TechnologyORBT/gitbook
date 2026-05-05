# Facilitator Operations

Once onboarded, a facilitator gains access to a **dedicated interface or API layer** that enables them to interact directly with the ORBT protocol within the bounds of their approved permissions and capacity. Through this interface, facilitators can execute a defined set of actions tied to their strategic operations and minting rights.

#### **1. Minting and Burning 0xAssets**

Facilitators are authorized to **mint or burn 0xAssets** within the **limits of their approved capacity**. These operations are conducted through designated facilitator contract functions (e.g., `mint(address,uint256)` for GHO-style implementations; ORBT follows a similar structure).

Facilitators may mint **0xUSD** or other assets directly to their operational address or to a **multisig wallet** they control, for subsequent deployment into approved strategies.\
Each minting event increases their **bucket usage level**, which is continuously tracked by the protocol to enforce capacity and risk constraints.

#### **2. Deploying and Operating Pockets**

Facilitators can **deploy new Pockets** or invoke existing **strategy templates** to execute opportunities. They typically operate **off-chain automation systems** — such as bots, keepers, or monitoring agents — that identify profitable conditions (e.g., price discrepancies across exchanges or funding rate arbitrage) and trigger Pocket deployments in response.

Facilitators may also execute **scheduled or recurring strategies**, such as liquidity rebalancing or yield redistribution. In this context, facilitators act as **strategy operators**, converting observed market signals into **ORBT intents** and executing them through the protocol’s infrastructure.

While ordinary users initiate Pockets through a user interface, facilitators interact programmatically — potentially managing **dozens of Pocket transactions** across chains in parallel.

#### **3. Adjusting Strategy Parameters**

Facilitators may adjust **strategy-level parameters** within the bounds defined by governance.\
For example, a facilitator managing an “institutional yield” strategy could dynamically select between external platforms such as **Compound** or **Aave** based on current yield rates.

However, facilitators cannot modify **core logic**, **risk parameters**, or **governance-defined constraints**. Any change that materially alters a strategy’s risk profile or operational scope requires formal governance approval. This safeguard ensures that facilitators cannot unilaterally escalate systemic exposure.

#### **4. Monitoring and Maintenance**

Facilitators are responsible for **ongoing monitoring and maintenance** of their active strategies and Pockets. They typically operate their own infrastructure — such as blockchain nodes, automation keepers, or off-chain services — to react to market conditions in real time.

Examples include:

* Triggering new Pockets when arbitrage opportunities appear.
* Adjusting or closing positions when collateral ratios or yields move unfavorably.
* Managing liquidity rebalancing or position exits during volatility events.

While the ORBT protocol implements **automated safeguards** at the system level, facilitators provide an additional **human and algorithmic oversight layer**, ensuring that operational execution remains aligned with performance goals and risk management standards.
