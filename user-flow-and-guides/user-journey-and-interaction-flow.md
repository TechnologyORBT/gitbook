# User Journey & Interaction Flow

## User Journey & Interaction Flow

From a user experience perspective, interacting with ORBT follows a structured sequence of actions.\
Whether the participant is a **retail DeFi user**, **developer**, or **institution**, the process remains largely consistent — with **facilitators** providing an additional operational layer for institutional access .

{% stepper %}
{% step %}
### Onboarding and Wallet Integration

Users begin by connecting to the **ORBT dApp** or **API** using a **web3 wallet** (e.g., MetaMask, Rabby, WalletConnect, etc.), or via a **custodial interface** if using institutional accounts.

Since ORBT is **multi-chain**, users can choose their preferred network — Ethereum, Arbitrum, Base, etc. — and ORBT’s backend automatically handles **cross-chain transfers and routing** where necessary.

* For institutional users, a **KYC or verification step** may be required, depending on ORBT’s compliance configuration (see §8).
* Retail DeFi users, however, can **interact permissionlessly** with all smart contracts.

The interface provides **real-time system metrics**, such as total value locked (TVL), collateral health, yield rates, active strategies, and protocol utilization.
{% endstep %}

{% step %}
### Depositing Assets

After onboarding, users can deposit assets to either **mint 0xAssets** or **directly invest** in strategies.

**Example:**

* _Alice_ deposits **10 ETH**. The interface shows her borrowing power and maximum mintable amount (e.g., she can mint up to 0xUSD 1000).
* _Bob_ deposits **100,000 USDC** directly through the UCE interface to obtain **100,000 0xUSD**, swapping one stablecoin for another with minimal fees.

All deposits are **confirmed on-chain**, and the **User Position Manager (UPM)** updates the user’s account state.

* Depositing collateral grants borrowing power in 0xUSD.
* Depositing stablecoins via UCE yields immediate conversion to 0xUSD.
* Users can optionally **stake ORBT tokens** in the **User Staking Module (USM)** to earn governance rewards — though this is entirely independent from liquidity functions.
{% endstep %}

{% step %}
### Choosing an Intent or Strategy

Once assets are deposited, users select a **strategy or intent** from a curated list of **Pocket templates** or craft a **custom strategy**.

Common examples include:

* “**Earn Savings Rate (Low Risk)**”
* “**Provide Liquidity on DEX (Medium Risk)**”
* “**Leveraged Yield Farm (High Risk)**”
* “**Cross-Chain Arbitrage (Facilitator-Run)**”

Each intent displays:

* **Estimated APY**
* **Risk level**
* **Fees and expected duration**

**Example:**

* Alice chooses _Stablecoin Savings (\~8% APY)_, which stakes her 0xUSD in ORBTMM or an external market to earn stable yield.
* Bob selects _ETH Liquid Staking_, using his ETH to mint 0xUSD, swap for staked ETH via UCE, and deploy into a yield protocol — all handled automatically by a Pocket.

Users can adjust parameters such as **allocation**, **duration**, and **stop-loss** thresholds through the UI.\
The system then bundles and prepares all necessary transactions for seamless execution.
{% endstep %}

{% step %}
### Authorization (Permit Layer)

When a user confirms a strategy, the system may require **token approvals** or **signature-based authorization** to permit funds usage.

Using **EIP-2612 permits**, users can grant spending permissions with a **single signature** rather than multiple transactions — reducing friction and gas costs.

**Example:**

* Alice signs a message allowing a Pocket contract to use up to `X` 0xUSD from her balance.
* The Pocket can now execute on her behalf without an extra `approve()` call.

This “**intent-based UX**” approach ensures users define _what they want_ (e.g., “earn yield”) rather than _how to do it_, improving simplicity and user trust.
{% endstep %}

{% step %}
### Execution of Strategy

Once authorized, ORBT executes the chosen intent.

If handled by a **facilitator**, requests are routed to their infrastructure to trigger required on-chain transactions.\
Otherwise, ORBT’s **core smart contracts** execute operations directly.

Typical flow includes:

1. **Allocating user assets** — taking the specified 0xUSD or collateral.
2. **Transferring liquidity** from ORBTMM into the appropriate Pocket.
3. **Executing Pocket logic** — swaps, lending, staking, or LP provisioning.
4. **Cross-chain operations** — handled automatically through secure bridge messages (LayerZero, CCIP, etc.).
5. **Result handling** — profits or yield-bearing tokens are held by the Pocket or custody modules on behalf of the user.

The UI updates to show:

* Strategy status (“In Progress”)
* Current yield or P/L metrics
* Real-time balance updates

Some strategies run continuously (e.g., auto-compounding yield farms), while others are **one-off executions** (like arbitrage trades).
{% endstep %}

{% step %}
### Monitoring and Managing

Active strategies are viewable in the **user dashboard** under “My Pockets.”

Example interface:

> _Pocket #7 – DEX Liquidity_\
> • Amount: 1,000 0xUSD\
> • Current Value: 1,020 0xUSD\
> • Profit: +2.0%

Users can:

* **Pause, withdraw, or redeem** profits,
* **Set risk controls**, like stop-loss or max duration,
* **View on-chain logs** and verify execution through ORBT’s **subgraph** or **API**.

ORBT maintains full **on-chain transparency** — users can verify each action independently.\
If markets shift or performance declines, facilitators or automated logic can **exit positions** early to preserve capital.
{% endstep %}

{% step %}
### Completion or Withdrawal

When users end a strategy or when it concludes automatically, the Pocket unwinds its positions:

* For a savings Pocket: funds and earned interest are withdrawn from lending markets.
* For liquidity Pockets: LP tokens are removed and converted back to 0xUSD or the target asset.

Returned funds are deposited back to the **UPM account** or the user’s wallet, depending on configuration.

If the strategy involved leverage or borrowing:

* The system **automatically repays** any outstanding debt,
* Returns the **net profit** to the user.

**Example:**\
Alice withdraws from her savings Pocket — her initial 0xUSD plus yield are credited back to her account.\
Bob closes his LP position — assets are unwound, converted to 0xUSD, and repaid into his vault.
{% endstep %}

{% step %}
### Closing Position (_Optional_)

To fully exit ORBT, users can **repay remaining debts** and **withdraw all collateral**.

**Example:**

* Alice repays her 0xUSD debt and unlocks her ETH from the **User Collateral Engine (UCE)**.
* Bob swaps his 0xUSD back to USDC via UCE and withdraws to his wallet.

Once collateral and debt positions are cleared, the **UPM position closes** completely, ending the user’s engagement with ORBT.
{% endstep %}

{% step %}
### Claiming Staking Rewards

Users staking ORBT in the **USM (User Staking Module)** can periodically claim accumulated rewards — distributed in **0xUSD**, **ORBT**, or a mix, as determined by governance.

**Example:**\
If protocol revenue was strong, Alice’s staked ORBT may have earned **50 0xUSD** over a month.\
Claiming triggers a transaction that transfers rewards from the module to her wallet.

If unstaking, there may be a **cooldown or vesting period**, mirroring established models (e.g., MakerDAO’s staking proposals), to prevent abuse.
{% endstep %}
{% endstepper %}

***

### Example Recap: End-to-End Flow

**Alice’s Journey:**

* Deposited ETH → Minted 0xUSD → Staked in savings → Earned interest → Withdrew seamlessly.

**Bob’s Journey:**

* Deployed capital in a complex facilitator-run yield strategy → Monitored performance via dashboard → Redeemed profits and closed his position.

Both users benefited from ORBT’s **intent-based automation**, which abstracts away complex multi-step operations.\
Liquidity routing, cross-chain interactions, and yield optimization all occur behind the scenes — efficiently and securely.

Throughout this process, users retain **non-custodial control** of their assets.\
Funds are always withdrawable unless engaged in a **time-locked strategy** (which is clearly indicated in advance).

The **permissioned aspect** applies only to which strategies are approved, not to user funds themselves.\
Governance retains the ability to **pause** specific operations during emergencies (see §7: Security), with built-in **timelocks and multisig oversight** ensuring full transparency and user protection.
