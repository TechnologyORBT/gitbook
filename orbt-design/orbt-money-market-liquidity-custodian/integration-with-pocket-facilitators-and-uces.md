# Integration with Pocket, Facilitators & UCEs

ORBTMM is deeply interconnected with other components, ensuring seamless liquidity orchestration:

#### **Pockets**

* Serve as the **custodial endpoints** of underlying stablecoins.
* ORBTMM interacts with them for withdrawals, Aave delegation, and reserve top-ups.
* Global and allocator-specific pockets together maintain the ecosystem’s settlement flexibility.

#### **Facilitators**

* Facilitators are authorized entities that execute **intent-based transactions** using delegated liquidity.
* When a facilitator receives an intent (e.g., settle cross-chain payment), ORBTMM allows instant borrowing through Aave credit delegation and repays shortly after from inflows.

#### **Unified Collateral Engine (UCE)**

* The UCE is the control plane — quoting, minting, and burning 0xAssets.
* It coordinates with ORBTMM to determine when to pull liquidity or rebalance reserves.
* UCE metrics (e.g., `reserveBps`, `debtIndex`, `utilization`) feed directly into ORBTMM’s automation layer.

### E2E Example

* A user mints $1M 0xUSD by depositing $1.5M ETH (150% CR).
* UCE keeps $250K in reserve, sends $750K to the Pocket.
* Pocket supplies to Aave → earns aUSDC yield.
* Later, a $600K redemption occurs: UCE pays $250K from reserve, $350K via allowance pull from Pocket.
* Meanwhile, a facilitator executes an intent needing $200K USDC — borrows via credit delegation, settles instantly, repays within 15 minutes.
* HF stays > 2.2; aUSDC continues accruing yield — liquidity restored.
