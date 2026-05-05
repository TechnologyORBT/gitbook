# Integration with Pocket, Allocators & UCEs

ORBTMM is deeply interconnected with other components, ensuring seamless liquidity orchestration:

#### **Pockets**

[**Pockets** ](../pocket/)act as the custodial endpoints for underlying stablecoins and serve as the first layer of capital control within ORBTMM.

* Each Pocket securely holds liquidity contributed by allocators or system reserves.
* ORBTMM interacts with Pockets to perform key functions such as:
  * Liquidity withdrawals
  * Credit delegation to external markets
  * Reserve top-ups and rebalancing operations
* Both **global** and **allocator-specific Pockets** maintain flexible liquidity pathways across the ecosystem, ensuring that settlements and rebalances occur smoothly and on demand.

This design isolates risk while providing structured access to liquidity for downstream components like the [**Unified Collateral Engine**](../uce/) **(UCE)** and [**Allocators**](../allocator/).

#### Allocators

[**Allocators**](../uce/concepts/allocators/) are authorized entities that execute **intent-based transactions** using delegated liquidity.\
They are critical for real-time, event-driven actions such as settlements, cross-chain payments, or liquidity redistributions.

When an allocator receives an intent (for example, _settle a cross-chain payment_):

1. ORBTMM enables **instant borrowing** from available liquidity via [**credit delegation**](../pocket/credit-delegation-for-instant-settlements/) to an integrated money market (such as Aave v3 or ORBTMM pools).
2. The allocator executes the transaction immediately.
3. The borrowed liquidity is repaid shortly afterward (typically from incoming user or protocol inflows).

This system ensures that liquidity is always **on call**, enabling instant settlements without requiring manual fund movements or rebalancing.

#### **Unified Collateral Engine (UCE)**

* The [UCE](../uce/) is the control plane: quoting, minting, and burning [0xAssets](../0xassets.md).
* It coordinates with ORBTMM to determine when to pull liquidity or rebalance reserves.
* UCE metrics (e.g., `reserveBps`, `debtIndex`, `utilization`) feed directly into ORBTMM’s automation layer.

### E2E Example

* A user mints $1M 0xUSD by depositing $1.5M ETH (150% CR).
* UCE keeps $250K in reserve, sends $750K to the Pocket.
* Pocket supplies to Aave → earns aUSDC yield.
* Later, a $600K redemption occurs: UCE pays $250K from reserve, $350K via allowance pull from Pocket.
* Meanwhile, allocator executes an intent needing $200K USDC, borrows via credit delegation, settles instantly, repays within 15 minutes.
* HF stays > 2.2; aUSDC continues accruing yield, liquidity restored.
