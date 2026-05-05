# How to buy 0xAssets

0xAssets represent on-chain, collateral-backed synthetic assets within the ORBT ecosystem.\
They can be acquired by swapping them directly via the ORBT protocol.

<figure><img src="../.gitbook/assets/buy 0xasset (1).png" alt=""><figcaption></figcaption></figure>

***

#### Using the dApp Interface

The user connects a supported Web3 wallet (MetaMask, Rabby, WalletConnect, or any compatible option) to the ORBT dApp.

After connecting, the user selects the **token they wish to swap** (for example, USDC, USDT, WETH, or WBTC) and enters the amount to exchange.

When the user confirms the transaction, the **User Collateral Engine (UCE)** detects the type of deposited token and executes a 1:1 swap into the corresponding 0xAsset.

After on-chain confirmation, the **User Position Manager (UPM)** updates the user’s balance to reflect the newly acquired 0xAssets.

***

#### Asset Families and Corresponding UCEs

Each **User Collateral Engine (UCE)** manages a distinct family of collateral assets and swaps its own 0xAsset variant:

| Asset Family       | Managed Collateral           | Swapped 0xAsset | Example Collateral |
| ------------------ | ---------------------------- | --------------- | ------------------ |
| **BTC Family UCE** | Bitcoin-backed assets        | `0xBTC`         | WBTC, cbBTC, tBTC  |
| **ETH Family UCE** | Ethereum-backed assets       | `0xETH`         | WETH, stETH, rETH  |
| **USD Family UCE** | USD stablecoin-backed assets | `0xUSD`         | USDC, USDT, DAI    |

When users deposit collateral into a specific UCE, it determines the corresponding 0xAsset they will swap.

Example:

* Alice deposits **WETH** → The **ETH Family UCE** swaps **0xETH**.
* Bob deposits **USDC** → The **USD Family UCE** swaps **0xUSD**.
* Charlie deposits **WBTC** → The **BTC Family UCE** swaps **0xBTC**.

***

#### Important to Note

* **Transparency:** All swap transactions are verifiable through ORBT’s subgraph and API.
* **Non-custodial architecture:** Users retain full ownership of assets; smart contracts only act per signed user instructions.
* **Collateral specificity:** Each 0xAsset type is swapped solely within its designated UCE family — cross-family collateral is not interchangeable.
