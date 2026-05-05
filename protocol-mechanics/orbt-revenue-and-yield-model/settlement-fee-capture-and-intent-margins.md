# Settlement Fee Capture & Intent Margins

Whenever a **Pocket** executes a trade or strategy, **ORBT can levy a fee**. This functions similarly to how exchanges charge trading fees or how Yearn applies performance fees.

For example, a **Pocket execution fee of 0.3%** on the transaction amount might be charged for a complex multi-step strategy, covering **gas costs and protocol usage**. Additionally, a **settlement fee** may apply on the outcome — such as **1% of profits** or **0.1% of the principal** upon successful completion.

These fees ensure that every use of ORBT’s infrastructure contributes to the **protocol treasury**.\
They can be **adjusted based on strategy type**:

* A **simple stable swap** via the UCE might incur only **0.05%**, keeping it competitive with DEXs.
* A **sophisticated cross-chain operation** could justify up to **0.5%**, reflecting higher complexity and resource usage.

Notably, these fees are typically **taken in-kind** (e.g., in **0xUSD** or whichever asset is traded) and are automatically **routed to the ORBT Treasury**.

By integrating fees directly at the **Pocket level**, ORBT generates revenue from both:

* **User activity** (individuals executing strategies), and
* **Facilitator activity** (facilitators also pay Pocket execution fees when deploying strategies).

This model encourages efficient strategy execution and discourages unnecessary transaction spamming while ensuring consistent revenue capture across all protocol usage.
