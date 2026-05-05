# Pocket

**Pockets** are special-purpose smart contract accounts (or designated addresses) where capital is deployed to execute predefined strategies. Each Pocket functions as an **isolated strategy container** with its own permissions, operational logic, and lifecycle controls.

When a facilitator or user initiates a strategy - for example, a yield-farming operation on **Uniswap**, a lending action on **Compound**, or a cross-chain arbitrage - the **ORBT Money Market** allocates **0xAssets** (such as 0xUSD or other tokens) to a Pocket. The Pocket then autonomously executes the required transactions according to its programmed rules and permissions.

#### **Permit Layer and Isolation**

Each Pocket includes a **permit layer**, which restricts its behavior to **pre-approved operations** only. For instance, a Pocket may be permitted to interact solely with specific whitelisted protocols or to return funds exclusively to the Money Market upon completion. This architecture mitigates risk by ensuring Pockets cannot be exploited to drain funds or perform unauthorized actions.

Functionally, Pockets act as **temporary, bounded vaults**:

* They hold only the capital necessary for their strategy.
* They exist only for the duration of that strategy.
* They return profits or remaining capital back to the **Money Market** or the user’s **UPM position** once execution completes.

This design minimizes exposure, since funds are never idle in a strategy contract, and maximizes security and capital efficiency.

#### **Composability and Governance**

Pockets are **composable** building blocks. A complex intent may involve multiple Pockets operating sequentially or in parallel - for example, one Pocket performing a token swap while another provides liquidity simultaneously.

All Pocket templates and their strategy logic must be **approved by ORBT governance** to ensure compliance and security. Community developers can propose new Pocket strategies — such as novel yield farms or on-chain derivatives — which, once vetted and approved, can be deployed by facilitators. This open-yet-governed model promotes ongoing innovation similar to **Yearn’s Vault Strategy System**, but with stronger oversight and access control.

#### **Clearing and Execution Role**

The term **OCH (ORBT Clearing Hub)** highlights the Pocket’s role as an **execution endpoint** or “mini clearing house.” Each Pocket settles user intents — such as swapping asset A for B, providing liquidity, or borrowing — by interfacing with external protocols and clearing results back to the originating user or module.

In traditional finance terms, Pockets resemble **temporary settlement accounts** created solely to fulfill specific orders. Conceptually, they parallel mechanisms used by **CoW Protocol** or **UniswapX**, where user intents are resolved off-chain and executed atomically on-chain. In ORBT’s architecture, the Pocket is the **on-chain executor** of an intent, ensuring atomic, transparent, and efficient strategy completion.

{% hint style="info" %}
**Secure Strategy Execution:** Pockets isolate risk at the strategy level. Each Pocket operates with limited permissions, executes only approved transactions, and automatically clears funds back to the core system, ensuring contained exposure and predictable execution.
{% endhint %}
