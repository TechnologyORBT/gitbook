# Operational Model

Each stable asset supported by ORBT (e.g., USDC, DAI, USDT) is managed through a **dual-layer liquidity model** combining on-chain reserves and Pockets.

#### **Direct Custody**

* Pockets are **non-custodial smart addresses** (EOA, multisig, or vaults) that directly hold assets.
* The **UCE** interacts with these pockets via ERC-20 `allowance` permissions — it can pull funds as needed for settlements or redemptions.

#### **Reserve Buffering**

* A **per-asset reserve** (typically 15–35%) is retained within UCE itself to cover small, frequent redemptions.
* Larger outflows are fulfilled through **pocket pulls**, ensuring redemptions remain instant even during bursts.

#### **Yield Routing**

* Any **non-reserved balance** in a Pocket is automatically deposited into **Aave v3**, earning yield through aTokens (e.g., aUSDC).
* These aTokens continuously accrue value, representing the underlying plus interest.

#### **Credit Delegation for Settlement**

* Pockets can delegate a portion of their **borrowable capacity** on Aave to pre-approved settlement actors.
* This lets **Facilitators** borrow stablecoins instantly (backed by Pocket collateral) and fulfill user intents without waiting for pull confirmations.

#### **Incentive Attribution**

* Each Pocket is linked to a **referral mapping**.\
  When users transact using a specific Allocator’s code, the liquidity is first routed through that Allocator’s Pocket.\
  This encourages Allocators to **pre-provision reserves**, since faster fulfillment earns them spreads and user attribution.
