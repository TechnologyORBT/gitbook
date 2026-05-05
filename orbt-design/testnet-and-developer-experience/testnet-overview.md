# Testnet Overview

Before interacting with **mainnet ORBT**, developers and users are encouraged to test on the **ORBT Testnet deployment**.\
ORBT likely deploys on one of **Ethereum’s testnets** (like _Goerli_ or _Sepolia_) and possibly an **L2 test deployment** (e.g., _Arbitrum Goerli_) to simulate cross-chain functionality.

### Testnet Addresses

Developer documentation lists the addresses of all deployed contracts on testnet, which generally mirror mainnet functionality.

**For example:**

* **0xUSD (Goerli)** — `0x111...`
* **ORBT Token (Goerli)** — `0x222...`
* **UCE, UPM, etc.** — single or multiple system addresses
* **Test Faucet Address** — optional for obtaining tokens

These addresses are made publicly available in the developer documentation for easy reference.

### Faucets

The ORBT team operates a **faucet** for distributing **testnet 0xUSD and ORBT tokens**.

Developers (or testers) can request, for example:

* 1000 test 0xUSD, and
* Some test ORBT tokens to experiment with.

Access may be provided via:

* A **web-based faucet interface**, or
* A **Discord bot command**.

Since 0xUSD is minted against collateral, the faucet can either:

* **Directly mint test 0xUSD** for distribution, or
* **Provide test collateral** (like _GoerliETH_) so users can mint 0xUSD themselves.

{% tabs %}
{% tab title="Example" %}
On Goerli, to get test 0xUSD, you can either claim some from our faucet or mint it yourself by depositing GoerliETH through our test UI.
{% endtab %}
{% endtabs %}

### Testnet Environment Features

The **testnet parameters** may be configured more flexibly than mainnet to streamline testing.

#### Examples:

* **Lower collateral ratios** — to simplify minting and experimentation.
* **Centralized governance controls** — allowing the team to quickly reset or upgrade parameters.
* **All-access facilitator mode** — where the team acts as the default facilitator, enabling developers to request Pocket execution whitelisting easily.

However, ORBT aims to simulate **realistic mainnet conditions**, so even the testnet strives to reflect production-like flows (albeit with worthless tokens).

### Integration Testing

Developers are encouraged to **thoroughly test their integrations** using the testnet setup.

**Examples:**

* Wallet developers should test **0xUSD deposits, withdrawals, and minting** flows.
* DApp builders should ensure **Pocket execution** and **strategy interactions** behave as expected.

ORBT may also provide **debugging utilities** for scenario simulation:

{% tabs %}
{% tab title="Example" %}
Simulate a market crash by calling `orbtDebug.setPrice(ETH, $100)` on the testnet debug contract to observe system behavior.
{% endtab %}
{% endtabs %}

These debugging hooks are purely for testing purposes and allow developers to validate how ORBT modules respond under stress.

### Testnet Explorer and Analytics

ORBT provides **exploration and analytics capabilities** for developers on testnet.

Developers can use:

* **Etherscan’s testnet explorers** (Goerli, Sepolia, etc.) to view transactions, or
* A dedicated **ORBT testnet dashboard** if provided by the team.

Additionally, a **subgraph** is often deployed for testnet, allowing developers to:

* Run **GraphQL queries**, and
* Simulate data indexing workflows before mainnet deployment.

This ensures that integrations relying on subgraphs, APIs, or data pipelines can be tested in a live-like environment.

### Beta Testing Program

ORBT may operate a **developer beta testing program** on testnet.

Participants can:

* Build and share **demo integrations** or **testing scripts**,
* Earn recognition, or
* Qualify for **mainnet ORBT token rewards** or **ecosystem features**.

{% tabs %}
{% tab title="Example Incentive" %}
Build a demo integration with ORBT on testnet and share it publicly — you may receive mainnet ORBT tokens or be featured as a launch partner.
{% endtab %}
{% endtabs %}

This program helps ORBT onboard early developers and validate new integrations collaboratively.

### Deployment on Multiple Chains

ORBT’s vision is **multi-chain interoperability**.\
While Ethereum mainnet serves as the **primary deployment**, the protocol also intends to launch on **L2 networks** (like _Base_ or _Arbitrum_) for scalability and efficiency.

#### Developer Documentation Notes:

* Testnets will exist for each major deployment.
* Example
  * ORBT is deployed on Arbitrum Goerli testnet at addresses X, Y, Z.\
    Bridging between testnets is currently simulated by manually minting tokens on each side.

For **mainnet**, the documentation will later cover:

* **Bridging procedures** using **CCIP**, or
* Official portals for moving **0xUSD** between networks (e.g., Ethereum ↔ Polygon).

### Upgrades and Testnet Maintenance

The **testnet environment** is typically **one version ahead of mainnet** — serving as a live staging ground for upcoming features.

For example:

* **ORBT v1.1** (introducing new asset types or modules) will first launch on testnet.
* Developers can test new integrations before production rollout.

Announcements about testnet resets or upgrades will be posted in:

* Developer documentation, and
* Official channels (Discord, Telegram, or governance forums).

If major breaking changes occur:

* A **fresh contract set** will be deployed, and
* The old testnet version will be **deprecated** (clearly marked in docs).

This ensures continuity while allowing iterative upgrades.

### Interoperability and Composability

ORBT emphasizes **DeFi composability**.\
All **ORBT assets (0xUSD, 0xBTC, 0xETH)** conform to the **ERC-20 standard**, ensuring seamless integration across wallets, dApps, and DeFi protocols.

#### Integration Notes:

* Adding ORBT assets to an application is as straightforward as integrating **DAI** or **WETH** — simply use the provided token addresses and metadata (logo, decimals, etc.).
* The **unique aspect** lies in how these tokens are **minted and managed** through the ORBT system.

Advanced developers can further explore:

* **Yield-bearing integrations**,
* **Pocket automation**, and
* **Cross-chain liquidity operations**, though these are optional and more sophisticated.
