# How to unstake 0xAssets

Unstaking 0xAssets enables users to redeem their yield-bearing tokens (s0xUSD, s0xETH, s0xBTC) back into the original underlying assets.

Through the ORBT dApp, users can initiate the unstaking process, trigger the cooldown period, and withdraw their 0xAssets after completion.

***

#### Using the dApp Interface

Users in permitted jurisdictions can unstake their 0xAssets directly via the ORBT interface.

The unstaking workflow is as follows:

1. **Connect Wallet:** The user connects their preferred Web3 wallet (e.g., MetaMask, Rabby, WalletConnect) to the ORBT app.
2. **Select Staked Asset:** Choose the yield-bearing token they wish to unstake — such as s0xUSD, s0xETH, or s0xBTC.
3. **Click “Unstake”:** Initiate the unstaking action, which triggers a wallet prompt to sign the transaction.
4. **Sign Transaction:** The user confirms the transaction, which starts the unstake process on-chain.
5. **Cooldown Period Begins:** Once the transaction is confirmed, the system enters a mandatory cooldown period before withdrawal becomes available.
6. **Withdraw Assets:** After the cooldown expires, the user returns to the interface, clicks **“Withdraw,”** and signs a final transaction to claim their 0xAssets.

Upon successful withdrawal, the user’s s0xAssets are burned, and the corresponding 0xAssets — including any accumulated yield — are transferred back to their wallet or UPM account.

***

#### Important to Note

* **Cooldown Duration:** Each staking pool defines its own cooldown period — typically **7 days** — before 0xAssets can be withdrawn.
* **Reward Accrual:** Rewards continue to accumulate until the unstake transaction is confirmed on-chain.
* **Redemption Rate:** The number of 0xAssets received upon withdrawal depends on the **current value of s0xAssets** at the time of redemption, not at the time of staking.
* **Non-Decreasing Value:** The redemption rate for s0xAssets never decreases, ensuring users only receive equal or greater value than their initial stake.
* **Transparency:** Users can monitor the cooldown status and unstaking transactions directly via the ORBT dApp dashboard or the on-chain subgraph.
