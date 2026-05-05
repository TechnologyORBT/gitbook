# System Overview

## ORBT Unified Liquidity Layer

The layer abstracts heterogeneous underlying assets into a standardized 18-decimal 0xAsset per family (USD, ETH, BTC). Settlement and routing logic reside in [OrbtUCE](uce/) (`src/uce/OrbtUCE.sol`).

Assets are registered with a [pocket](pocket/) (vault address) and a reserve basis points (bps). Inbound transfers are split between contract-held reserves and pocket-deposited amounts. This provides always-on settlement capacity while keeping most liquidity productively allocated.

**Family Model:** A [UCE ](uce/)instance is configured for a [single AssetFamily](uce/concepts/swaps.md#instrument-classes) (BTC, ETH, USD) and its paired [0xAsset](0xassets.md). Family mismatches are rejected upon asset registration, enforcing strong isolation across families.

***

## 0xUSD and 0xAssets

[0xAsset](0xassets.md) (`src/token/base/0xAsset.sol`) is an ERC20 + ERC20Permit with owner-governed minter list and fixed 18 decimals. It has no cap and relies on governance/design-time invariants for supply discipline.

**Concrete wrappers:** 0xUSD, 0xETH, 0xBTC are thin 0xAsset specializations binding names/symbols and initial minter/owner setup.

**Mint/Burn Policy:** Only minters (e.g., UCE) can mint/burn. Users cannot mint directly. Redemptions and swaps rely on [UCE flows](uce/architecture.md#mint-side-a-0x).

***

## Unified Collateral Engine

**Overview**: Single source of truth for pocket routing, swaps, allocator state, credit ceilings/daily caps, referral codes, reserve policy, and dynamic redemption fees.

**Deployments**: One UCE per family; each is constructed with an immutable [contractFamily](uce/concepts/swaps.md#instrument-classes) and later wired with its corresponding 0xAsset via set 0xAsset.

### Concepts

#### [**Architecture**](uce/architecture.md)

* **Roles**: ADMIN and SIGNER (EIP-712 multisig).&#x20;
  * ADMIN manages operational settings; SIGNER signs time-locked actions. Thresholds are configurable within bounds via signed and queued actions with signer uniqueness and replay protection.
* **Pockets**: Per-asset vault addresses (`pockets[asset]`). Admin can set/migrate global pockets; allocators may set their per-asset pocket override. Migration logic uses allowance-bounded transferFrom to move balances; reverts on insufficient allowance or missing pocket.
* **Reserves and on-hand settlement**: Per-asset reserveBps retains a fraction of inbound on UCE (`reservedUnderlyingGlobal`) for instant settlement; `_pushAssetFromContext` consumes on-hand and reserved before pulling from pockets.
* **Linked list of allocators**: Minimal approximate ordering with `_rebalanceAllocatorUp/Down`; bounded traversal (e.g., 30 steps) protects from OOG in shortfall routines.
* **Pausing**: Both global pause and per-asset pause exist; swaps check both legs for per-asset paused state.

#### [**Swaps (all topologies)**](uce/concepts/swaps.md)

* **Supported paths**
  * 0x <-> external assets
  * 0x <-> s0xAsset
  * external <-> 0x
  * s0xAsset <-> 0x
  * Asset-to-same-asset and unsupported pairings revert.
* **Exact-In**
  * `swapExactIn(assetIn, assetOut, amountIn, receiver, referralCode)` preview driven. Inbound external assets are pulled, reserve applied, and remainder deposited to the selected pocket (referral-aware). Outbound 0x draws from allocator reserved inventory first; if unreserved inventory is insufficient, UCE mints 0x covering shortfall and updates dynamic redemption rate + global debt reconciliation.
* **Exact-Out**: `swapExactOut` mirrors the logic with max slippage guard on `amountIn.`
* **s0xAsset:** ERC4626 conversions via `convertToShares`/`convertToAssets` when 0x is counter-asset.
* **Allowed/Disallowed matrix**:&#x20;
  * Allowed {0x >external, external >0x, 0x >s0x, s0x >0x};&#x20;
  * Disallowed {same-asset, external↔external without 0x, allocator 0x >non-s0x}.&#x20;
  * Allocators must swap underlying when burning 0x to prevent bypassing inventory accounting.
* **Fee/redemption dynamics:** If unreserved 0x inventory is insufficient, UCE mints 0x to receiver and updates `baseRedemptionRate` with decay and redeemed fraction; fee is capped at 50% of shortfall and `totalAllocatorDebt` is written down by net redemption.
* **Reentrancy/pausing:** Swaps are non-Reentrant and gated by global `whenNotPaused` plus per-asset `assetPaused[asset]` for both legs.
* **Errors and liveness:** Reverts on invalid assets/pairs, paused assets, settlement mismatches for s0x paths, insufficient allocator reserved 0x, and insufficient pocket liquidity when pulling underlying.

#### [**Allocators**](uce/concepts/allocators/)

* Permissioned actors with line-of-credit to mint 0x inventory for market-making, liquidity orchestration, and settlement assurance. AllocatorState: allowed, debt, reserved0x, borrowFeeBps, per-asset pockets, line (ceiling, dailyCap, mintedToday, lastMintDay), referralCode.
* **Credit Minting:** allocatorCreditMint(allocator, amount) mints 0x to UCE custody, increments allocator debt and reserved inventory, enforces daily cap/ceiling, and rebalances allocator ranking.
* **Repayment:** allocatorRepay(asset, assets) pulls underlying to UCE, applies borrow fee in-kind to treasury, converts principal to 0x-equivalent (decimal normalized), and reduces allocator debt with global reconciliation.
* **Referral Attribution:** Each allocator optionally has a deterministically generated referralCode. User swaps with that code consume 0x from the allocator’s reserved inventory and route outbound/inbound pocket flows accordingly.
* **Pocket Overrides:** Allocators can set per-asset pockets (and migrate balances subject to allowances) via admin/multisig or self-service (setMyPocket).

#### [0xAssets Credit Minting](uce/concepts/allocators/0xasset-credit-minting.md)

Strictly allocator-initiated or ADMIN-triggered for an allocator. Users cannot mint.

* [**Debt-based accounting**](uce/concepts/allocators/debt-accounting-with-debt-index.md): Every credit mint increases allocator debt and totalAllocatorDebt. Reserved 0x is debited as redemptions occur.

#### [Referral Attribution and Inventory Consumption](uce/concepts/allocators/referral-attribution-and-inventory-consumption.md)

If referral maps to an allocator, 0x debits come from that allocator’s reserved inventory; otherwise, unreserved on-hand inventory is used, and shortfalls trigger mint+fee dynamics.

#### [Allocator Repayment](uce/concepts/allocators/allocator-repayment.md)

* **Fee Path**: Fee is skimmed to the treasury in underlying units; principal reduces debt according to 0x-equivalent normalization.
* **Shortfall Coverage:** Debt list rebalance ensures highest-debt allocators are prioritized when reconciling in stress events (bounded steps to avoid OOG).

#### [Setting up Allocators](uce/concepts/allocators/setting-up-allocators.md)

* Admin or timelocked multisig uses \
  `setAllocatorSingleByAdmin`/`setAllocatorSingleMultiSig` with operations: set/update allowed, line, pockets, borrow fee, referral code.
* Credit constraints enforce ceiling >= debt and valid daily caps before activation.

#### [Dynamic Redemption Fees](uce/concepts/dynamic-redemption-fees.md)

* **Mechanism**: `baseRedemptionRate` (1e18 precision) decays over time via `DECAY_CONSTANT` and increases with redemptions proportionally to redeemed fraction of 0x supply, bounded by `MAX_REDEMPTION_RATE.`
* Application: On 0x shortfall during redemptions, fee is computed, capped at 50% of shortfall, 0x is minted to receiver, and `totalAllocatorDebt` is proportionally written down by the net redemption amount.

#### [Reserve Policy](uce/concepts/reserve-policy.md)

* Default `RESERVE_BPS` = 2500 with per-asset overrides via `setAssetReserveBps`. Reserve is accounted on inbound non-0x flows and consumed first during settlements. Calibrate reserves against expected redemption flow and pocket liquidity.

#### [Vault State Management and Accounting](uce/concepts/state-management-and-accounting.md)

* Decimal normalization to a standard 18 decimals (`_normalizeToStandard`/`_normalizeFromStandard`) ensures consistent credit line and redemption accounting across heterogeneous tokens.
* Preview functions reflect exact swap and conversion math, ensuring deterministic UI/SDK estimates.

#### 1:1 Asset Model

No price oracle dependency in UCE; 1:1 normalization by decimals and direct 0x/s0x conversions uphold the par model. External price risk is isolated to pockets/strategies.

#### Adding Assets

Admin-only `addAsset(asset, family, pocket, reserveBps)`. Family must match, pocket must be non-zero, and bps must be <= 10,000.

#### Governance

* EIP-712 Action struct: `Action(bytes32 actionType, bytes32 payloadHash, uint256 nonce)`. Queue with actionTimeLock, execute post-ETA, single-use digests via usedActionDigests. Signer thresholds \[`minSignatures`, `maxSignatures`] enforced; duplicates rejected.
* Actions: thresholds updates, allocator config (allowance/line/fee/pockets/referral), per-allocator pocket setting, timelock updates.

#### Roles

ADMIN and SIGNER across UCE and s0xAsset. ADMIN configures operational parameters; SIGNER participates in multisig governance flows.

#### Signer Thresholds

UCE: `minSignatures`/`maxSignatures` default to 1/9; mutable via timelocked signed actions. s0xAsset defaults to 7/9 and is adjustable via signed action.

#### Queuing Actions

UCE supports queueAction to register EIP-712 digests with actionTimeLock delay. Replays prevented by usedActionDigests and presence in queuedActionEta.

#### Timelock

`actionTimeLock` enforces delay between queuing and execution. Adjustable by `ADMIN` or signed action.

#### Action Execution

`setThresholds`, `setAllocatorSingleMultiSig`, `setAllocatorPocket`, `setActionTimeLockSigned` all enforce queued digest consumption, uniqueness, and signature threshold checks.

#### Replay Protection

* Signed digests are single-use; duplicates are rejected. Signer uniqueness enforced per action to prevent duplicate signer counting.

***

## Pockets (include Migration)

* [Pockets](pocket/) are ERC-20 holding addresses used to warehouse liquidity for each registered asset. Global pocket per asset plus optional per-allocator override.
* Migration and pulls are allowance- and balance-bound: reverts with NoPocket or InsufficientPocketLiquidity (or InsufficientAllocatorPocketLiquidity for allocator pockets). Operationally, maintain adequate allowances from pockets to UCE/strategies.
* Reserve portion remains on [UCE ](uce/)for instant settlement; the remainder is pushed to pockets to be used by strategies.

***

## Allocators: Credit initiation and Repayment

* Initiation: Allocator or ADMIN triggers `allocatorCreditMint`. Enforces daily cap/ceiling, increments debt and reserved inventory, and reorders allocator position.
* Repayment: Allocator sends underlying, fee-in-kind to treasury, principal normalized to 0x; debt and global totals reduced accordingly.

***

## Credit Caps

* Two-tier controls: ceiling (absolute outstanding cap) and dailyCap (mint per 24h). A day roll resets mintedToday. Mints revert if they exceed caps.
* Daily cap window keyed by UTC day index (block.timestamp / 1 days); mintedToday resets when day rolls. Mints enforce both dailyCap and ceiling >= new debt.

***

## Global vs Allocator Liquidity

* Global liquidity: On-hand 0x and reserved underlying (reservedUnderlyingGlobal) at UCE contract level. Used for settlements without tapping pockets.
* Allocator liquidity: reserved0x held against allocator debt, consumed preferentially when swaps are attributed to the allocator via referral.

***

## Dynamic redemption fees

Time-decaying base rate updated on shortfall events; incentivizes allocators to maintain sufficient reserved 0x and reduces moral hazard by socializing redemption costs via debt write-down.

***

## Debt amortizations

Debt is amortized implicitly through repayments and netting during shortfall coverage where allocator list is rebalanced and totalAllocatorDebt is reduced by redemptions.

***

## Treasury Flows and Economic Incentives

* Fees: Borrow fee (bps) paid in-kind on repayments to treasury. Redemption fee manifests as 0x supply expansion and global debt write-down (not treasury income).
* Incentives: Allocators earn spread from market-making and referral attributions; stakers earn rewards (s0xAsset emissions and/or ORBT). Governance can adjust borrow fee and reserve bps per asset.

***

## [User Position Manager (UPM)](upm/)

### Arbitrary execution and safety

* Minimal orchestrator (`src/upm/OrbtUPM.sol`) supporting `doCall` and `doBatchCalls` using `Address.functionCall`, forwarding revert reasons and returning raw bytes. No built-in allowlist; gate usage at integration layer and minimize approvals.
* Inherits AccessControl/ReentrancyGuard/Pausable but current call functions are ungated; consider surrounding infra (frontends/relayers) for access policy. Keep [UPM ](upm/)stateless and avoid custody.
* Batch behavior is atomic per-call: each subcall reverts if its target reverts; there is no partial success aggregator.

### Strategies

* [Aave ](pocket/aave-only-pocket-strategy/)Supply Only Strategy: Stateless adapter (`src/strategy/AaveSupplyOnlyStrategy.sol`) that supplies ERC20 into Aave on behalf of a pocket. aTokens accrue to the pocket, preserving collateral/credit delegation. Only [UPM](upm/) may call.
* Funding: `supplyFromPocket` may pull shortfall from pocket (requires allowance) then approve and deposit to Aave for pocket; `supplyFromSelf` uses adapter’s balance. Approvals are set per call and bounded to amount.
* Operational: Keep adapters stateless; receipts minted to pockets; ensure allowances are narrowly scoped and revoked when practical. Extend with more adapters following this pattern (LST/LRT, RFQ settlement).

### Money Market Supply

Conservative base-yield capture with instant settlement via aToken receipts to pockets; compatible with [credit delegation.](pocket/credit-delegation-for-instant-settlements/)

### [Pocket](pocket/)

Pocket is the beneficiary of strategy receipts; manage allowances from pocket to UPM/strategy and monitor balances/allowances to prevent liquidity stalls.

### \<more additions on future strategies>

Leverage-supply, delta-neutral hedging, cross-chain routers; maintain UPM-only gating and idempotence.

### [User Savings Module](usm/)

* 0xAsset deposits
  * s0xAsset (`src/usm/s0xAsset.sol`) wraps a [0xAsset](0xassets.md), maintaining `exchangeRateRay` with RAY math; accrues per-second via `_rpow(rateRay, dt, RAY)` and mints underlying to vault equal to  $$totalSupply * (Δrate) / RAY$$.
* ERC4626 accrual
  * Deterministic previews for deposit/mint/withdraw/redeem; `maxDeposit`/`maxMint` unlimited; `maxWithdraw`/`maxRedeem` reflect balances and projected rate.
  * Exit buffer (`exitBufferBps`) caps per-tx withdrawals as a fraction of `totalAssets()`, to align with [UCE ](uce/)reserve and pocket liquidity.
* Reward emission
  * Prefunded reward token with vault allowance; `rewardIndexRay` accrues as \
    $$(dt * rewardRatePerSecond) * RAY / totalSupply (guarded when supply==0)$$\
    Per-user deltas recorded; claim pulls from vault.
* Redeeming Constraints
  * `minUnstakeDelay` enforces per-recipient lock on incoming shares (mints/transfers); UIs should display `earliestUnstakeTime`.
* Governance
  * EIP-712 signed actions (rate, reward config, rewards-only mode, thresholds) with replay protection and signer uniqueness. Rewards-only mode pins `exchangeRateRay` and resets `rateRay` to `RAY` if needed.

***

## [Rewards](../protocol-mechanics/savings-rate.md#orbt-savings-rate)

* Savings Rewards
* **Contract:** `src/rewards/StakingRewards.sol` _(Synthetix-style)_. Tracks deposit balances, reward per token, and vesting windows. Reentrancy-guarded.
* **Emission math:**
  * `rewardPerToken = rewardPerTokenStored + (min(block.timestamp, periodFinish) - lastUpdateTime) * rewardRate * 1e18 / totalSupply` when `totalSupply > 0`.
  * **User earnings:** `earned = balance * (rewardPerToken - userPaid) / 1e18 + rewards[user]`.
* **Lifecycle:**
  * `notifyRewardAmount(reward)` rolls periods and ensures `rewardRate <= balance / rewardsDuration`.
  * `setRewardsDuration` updates duration with leftover handling.
* **Roles:**
  * Owner can set `rewardsDistribution` and `rewardsDuration`.
  * `rewardsDistribution` funds new periods.
  * Recover function prevents recovering deposited token.
* **Edge cases:**
  * Zero deposit or withdraw reverts.
  * Referral event on `stake(amount, referral)` is informational only and does not affect accrual.
* **Reward Accumulation**
  * s0xAsset rewards index accrues per-second and is previewable; `StakingRewards` accrues per-block-time and exposes `lastTimeRewardApplicable`, `getRewardForDuration` for UI.
