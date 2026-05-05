# Operational Model

Each stable asset supported by ORBT (e.g., USDC, DAI, USDT) is managed through a **dual-layer liquidity model** combining on-chain reserves and Pockets.

#### **Direct Custody**

* Pockets are **non-custodial smart addresses** (EOA, multisig, or vaults) that directly hold assets.
* The [**UCE** ](../uce/)interacts with these pockets via ERC-20 `allowance` permissions, it can pull funds as needed for settlements or redemptions.

#### **Reserve Buffering**

* A [**per-asset reserve**](../uce/concepts/reserve-policy.md) (typically 15–35%) is retained within UCE itself to cover small, frequent redemptions.
* Larger outflows are fulfilled through **pocket pulls**, ensuring redemptions remain [instant](../../security-and-safety/capital-protection.md#reserve-buffer-and-instant-settlement) even during bursts.

#### **Yield Routing**

* Any **non-reserved balance** in a Pocket is automatically deposited into ORBTMM, Aave v3 or other vetted money markets, earning yield through their interest-bearing tokens (e.g., aUSDC).
* These aTokens continuously accrue value, representing the underlying plus interest.

#### **Credit Delegation for Settlement**

* Pockets can delegate a portion of their **borrowable capacity** on Aave or others to pre-approved settlement actors.
* This lets **allocators** borrow stablecoins instantly (backed by Pocket collateral) and fulfill user intents without waiting for pull confirmations.

#### **Incentive Attribution**

* Each Pocket is linked to a **referral mapping**.
* When users [transact using a specific Allocator’s code](../uce/concepts/allocators/referral-vs-non-referral.md), the liquidity is first routed through that Allocator’s Pocket.
* This encourages Allocators to **pre-provision reserves**, since faster fulfillment earns them spreads and user attribution.
