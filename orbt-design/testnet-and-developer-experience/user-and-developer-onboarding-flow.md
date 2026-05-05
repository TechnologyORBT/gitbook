# User & Developer Onboarding Flow

Developers primarily interact with ORBT via **smart contract calls**.\
All ORBT smart contracts are designed with **standard interfaces** where possible and are **publicly documented**.

### 0xUSD and 0xAsset ERC-20 Tokens

These implement the **ERC-20 standard** (with extended **permit functionality EIP-2612**).\
Developers can treat `0xUSD` like any other stablecoin token contract — functions such as `transfer()`, `approve()`, and `permit()` are available.

The addresses of 0xUSD on each chain are published in the documentation (e.g., on Ethereum mainnet `0xUSD` might be at address `0x...`).

**Example Usage:**

```solidity
0xUSD.transfer(recipient, amount);
```

Transfers stablecoin tokens to a recipient.

Or using `permit()` for meta-transactions:

```solidity
0xUSD.permit(owner, spender, value, deadline, v, r, s);
```

> Documentation includes snippets explaining correct domain separator usage for EIP-2612 permits.

### ORBT Money Market (Liquidity Pool) API

The **ORBTMM contract** (analogous to a liquidity pool or controller) exposes functions for **depositing, withdrawing**, and **borrowing** liquidity — primarily for Pocket strategies.

#### Core Functions

```solidity
function deposit(uint256 amount, address onBehalfOf)
```

A user deposits 0xUSD (after approving).\
Returns a receipt token or updates internal accounting for user shares.

```solidity
function withdraw(uint256 amount, address to)
```

Withdraws liquidity from the pool to a specified address.

```solidity
function requestPocketLiquidity(address pocket, uint256 amount)
```

Called internally by the Pocket Factory when executing an intent, transferring funds to the Pocket under defined conditions.

These functions enable other DeFi platforms to integrate ORBTMM as a **yield source** — e.g., wallets could display “Deposit into ORBT and earn X%”.

### Pocket Factory / Execution Interface

ORBT’s modular architecture supports **Pocket creation and execution** for strategy automation.\
A **Pocket Factory contract** deploys new Pocket instances for predefined or custom strategies.

#### Example Functions

```solidity
function createPocket(bytes32 strategyType, bytes calldata params)
```

Instantiates a new Pocket contract using a defined strategy template.

```solidity
function executePocket(address pocket)
```

Triggers execution of a Pocket (if not automated).

While many Pockets operate autonomously after creation, developers integrating custom strategies can invoke these functions directly.\
For example, a DEX aggregator may call a “Swap Pocket” to execute a large trade using ORBT liquidity.

### User Position Manager (Vault) API

If ORBT allows **direct vault-level management** (like opening a collateralized position), developers can interact with the **User Position Manager (UPM)** contract.

#### Example Functions

```solidity
function openPosition(address collateralType, uint256 collateralAmount, uint256 debtAmount)
```

Deposits collateral and borrows (mints) 0xUSD in one transaction.

```solidity
function closePosition(uint256 positionId)
```

Closes or repays an existing vault position.

Equivalent flows may also use `repay()` or `withdraw()` individually.\
These internally invoke UCE to **lock collateral** and **mint/burn 0xUSD**.

{% hint style="info" %}
A yield aggregator could allow users to borrow 0xUSD automatically for leveraged yield strategies.
{% endhint %}

### Governance and Staking Contracts

ORBT governance contracts follow standard frameworks such as **Governor Bravo**, exposing familiar voting interfaces.\
While not all developers need these directly, they are essential for governance tooling or analytics integrations.

#### Example Governance Functions

```solidity
function propose(...)
function castVote(uint256 proposalId, uint8 support)
```

#### Staking Module

```solidity
function stake(uint256 amount)
function unstake(uint256 amount)
function claimRewards()
```

Stakers earn 0xUSD or ORBT rewards, which can be claimed periodically.\
These contracts align governance and participation incentives across the ecosystem.

### UCE (Unified Collateral Engine) API

The **Unified Collateral Engine(UCE)** provides seamless 1:1 swaps between 0xUSD and other stablecoins (e.g., USDC).

#### Example Function

```solidity
function swapStable(address inputToken, uint256 inputAmount) returns (uint256 outputAmount)
```

Used for stablecoin conversions, e.g., swapping **USDC → 0xUSD** or vice versa at near-par rates.\
Developers can integrate this into DEX UIs, arbitrage bots, or portfolio tools.

The documentation will specify:

* Current **swap fee** (e.g., 0.1%), and
* Any **transaction or size limits**.

{% hint style="info" %}
MakerDAO’s `buyGem()` and `sellGem()` analogies apply here.
{% endhint %}

### Access Control

ORBT uses a robust **role-based access control (RBAC)** model.\
All user-facing functions are public/external, while privileged operations are restricted to governance.

#### Example

```solidity
function addFacilitator(address fac, string label, uint128 cap)
```

Only callable by the `FACILITATOR_MANAGER_ROLE` (governance).

{% hint style="warning" %}
Developer docs highlight admin-only functions to prevent accidental misuse in dApps.
{% endhint %}

### Libraries and SDKs

To simplify integration, ORBT provides both **on-chain libraries** and **off-chain SDKs**.

#### JavaScript / TypeScript SDK

Available as an NPM package — for example:

```bash
npm install @orbt-protocol/sdk
```

**Usage Example:**

```typescript
import { Orbt } from "@orbt-protocol/sdk";

const orbt = Orbt.connect(provider);
await orbt.pocket.createStrategy("yield_farm", params);
```

The SDK abstracts contract interactions, handles multi-chain configurations, and integrates standard functions (e.g., deposits, swaps, Pocket creation).

### Subgraph and Data API

ORBT deploys a **TheGraph subgraph** (or similar indexer) to expose key protocol data for querying.

**Example Queries:**

* User positions by address
* Total 0xUSD supply
* List of facilitators and liquidity usage

**Example GraphQL Query:**

```graphql
query {
  positions(where: { owner: "0xabc..." }) {
    collateralType
    collateralAmount
    debtAmount
    healthFactor
  }
}
```

{% hint style="info" %}
The subgraph endpoint and schema are documented, providing developers real-time analytics access (noting the usual eventual consistency caveat).
{% endhint %}

### REST API (Off-Chain Data)

In addition to subgraphs, ORBT or third-party providers may offer a **RESTful API** that aggregates:

* Live APYs and yield rates,
* Protocol statistics, and
* Historical performance data.

Preference remains for **trustless, on-chain** or **subgraph-based** sources, but REST APIs support convenient integration for wallets and dashboards.

### Example Code Snippets

#### Swapping Stablecoins via ORBT UCE

```javascript
const ORBITUCE = new ethers.Contract(ORBITUCE_ADDRESS, ORBITUCE_ABI, signer);
await USDC.approve(ORBITUCE.address, amount);
const tx = await ORBITUCE.swapStable(USDC.address, amount);
const receipt = await tx.wait();
console.log("Received 0xUSD:", receipt.outputAmount);
```

#### Opening a Leveraged Yield Position

```javascript
await ETH.approve(ORBITUCE_ADDRESS, collateralAmount);
await UPM.openPosition(ETH.address, collateralAmount, debtAmount);
await PocketFactory.createPocket("investStrategy", { positionId });
```

#### Querying User Positions via Subgraph

```graphql
query {
  positions(where: { owner: "0xabc..." }) {
    collateralType
    collateralAmount
    debtAmount
    healthFactor
  }
}
```
