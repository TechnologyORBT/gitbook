---
description: >-
  USM (User Savings Module) offers two plug‑and‑play savings primitives that
  integrators and farmers can compose with: s0xAsset and ORBT StakingRewards.
  This page explains how to integrate.
---

# Integration & Usage guide

### High level architecture

```markup
User/Integrator Wallet
├─ deposits 0xAsset ──► s0xAsset (ERC4626 vault)
│ • exchangeRateRay grows per second (when enabled)
│ • optional rewardToken emissions via rewardIndexRay
│ • withdrawals can be time‑gated (minUnstakeDelay) and capped (exitBufferBps)
│
└─ stakes ORBT ──► ORBT StakingRewards
• linear emissions over rewardsDuration
• claim via getReward(); combined exit()
```

### Quick start

#### A) Integrate [**s0xAsset**](../../user-flow-and-guides/how-to-buy-s0xasset.md) (ERC‑4626 vault)

1. **Approve & deposit**
   * User approves `underlyingAsset` to the s0xAsset contract
   * Call `deposit(assets, receiver)` or `mint(shares, receiver)`
2. **(Optional) Read APY / conversion previews**
   * `previewDeposit`, `previewMint`, `previewWithdraw`, `previewRedeem`
   * `previewExchangeRateRay()` to read up‑to‑date exchange rate without state changes
3. **Withdraw or redeem**
   * Call `withdraw(assets, receiver, owner)` **or** `redeem(shares, receiver, owner)`
   * Enforced:
     * `minUnstakeDelay` (if set) - withdrawals/redeems require the recipient to be past the lock
     * `exitBufferBps` - caps max assets withdrawable per call vs vault liquidity
4. **Rewards (if configured)**
   * Read `claimable(account)`
   * Claim via `claim(to, amount)` (use `0` to claim all)

> Note: `drip()` can be called permissionlessly to settle interest & rewards. Integrators generally **do not** need to call it in UI flows; use `preview*` reads.

#### B) Integrate **ORBT StakingRewards**

1. **Approve & stake**
   * User approves `ORBT` to the staking contract.
   * Call `stake(amount)` (or `stake(amount, referral)` if you track referrals)
2. **Claim rewards**
   * Call `getReward()`; or use `exit()` to withdraw entire stake and claim in one call
3. **Withdraw**
   * Call `withdraw(amount)` to unstake ORBT (no lock built into this contract)

> Reward epochs are funded by `rewardsDistribution` via `notifyRewardAmount(reward)` and stream linearly for `rewardsDuration`.

### Behavior & math

#### s0xAsset: interest & shares

* **Exchange rate accumulator**
  * `exchangeRateRay` (assets per share, scaled 1e27) grows per second by `rateRay` via `drip()`.
  * If `rewardsOnlyMode = true` (default), the exchange rate is **frozen** (no interest minting). Governance can later enable interest by turning this off and (optionally) setting `rateRay`.
* **Interest minting**
  * On growth, the vault mints additional underlying to itself: `interestAssets = totalShares * (newRate - oldRate) / RAY`.
  * **Assumption**: `underlyingAsset` must expose `mint(address,uint256)` and grant minter rights to s0xAsset.
* **Rewards index**
  * `rewardIndexRay` accumulates rewards per share (`RAY` scale). On user balance changes, per‑account debt is advanced to keep rewards accurate.
* **Withdrawal guardrails**
  * `minUnstakeDelay` - users receiving/minting shares have their `earliestUnstakeTime` pushed forward; withdrawals/redeems require `block.timestamp >= earliestUnstakeTime[owner]`.
  * `exitBufferBps` - caps `withdraw(assets, ...)` to a fraction of `totalAssets()` (e.g., 10\_000 = 100%).

**APR estimator (interest):**

$$
\displaystyle \text{APR} \approx \left( \frac{\text{rateRay}}{10^{27}} \right)^{\text{secondsPerYear}} - 1
$$

**APR estimator (rewards):**

$$
\displaystyle 
\text{rewardsPerYear} = \text{rewardRatePerSecond} \times \text{secondsPerYear}
$$

$$
\displaystyle 
\text{APR}_{\text{rewards}} \approx \frac{\text{rewardsPerYear} \times \text{rewardTokenPrice}}{\text{TVL}_{\text{in underlying}}}
$$

ℹ️ _(Price/TVL come from off‑chain sources.)_

#### ORBT StakingRewards - emissions

* `notifyRewardAmount(reward)` sets `rewardRate` for the next epoch:
  * If previous epoch ended: `rewardRate = reward / rewardsDuration`
  * If mid‑epoch: carry over leftover and smooth: `rewardRate = (reward + remaining * oldRate) / rewardsDuration`
* View helpers:
  * `rewardPerToken()` and `earned(account)` (standard Synthetix math)
  * `getRewardForDuration() = rewardRate * rewardsDuration`

**APR estimator:**

$$
\displaystyle 
\text{rewardsPerYear} = \text{rewardRate} \times \text{secondsPerYear}
$$

$$
\displaystyle 
\text{APR} \approx \frac{\text{rewardsPerYear} \times \text{rewardTokenPrice}}{\text{totalStakedValue}}
$$

***

### Frontend

#### Read user balances & claimables

```solidity
// s0xAsset
uint256 shares = sZeroXAsset.balanceOf(user);
uint256 assets = sZeroXAsset.convertToAssets(shares);
uint256 claimable = sZeroXAsset.claimable(user);

// ORBT StakingRewards
uint256 staked = stakingRewards.balanceOf(user);
uint256 pending = stakingRewards.earned(user);
```

#### Approve & deposit / stake (ethers.js)

```typescript
// s0xAsset deposit
await underlying.approve(sZeroXAsset.address, amount);
await sZeroXAsset.deposit(amount, user);

// ORBT stake
await orbt.approve(stakingRewards.address, amount);
await stakingRewards.stake(amount);
```

#### Withdraw / redeem (ethers.js)

```typescript
// s0xAsset redeem exact shares
await sZeroXAsset.redeem(shares, user, user);

// ORBT withdraw
await stakingRewards.withdraw(amount);
```

#### Claim rewards (ethers.js)

```typescript
// s0xAsset
await sZeroXAsset.claim(user, 0); // 0 = claim all

// ORBT StakingRewards
await stakingRewards.getReward();
```

#### Preview UX (no state changes)

```typescript
const sharesOut = await sZeroXAsset.previewDeposit(assets);
const assetsOut = await sZeroXAsset.previewRedeem(shares);
const rateRay = await sZeroXAsset.previewExchangeRateRay();
```

***

### Governance & admin surfaces

#### s0xAsset

* **AccessControl `ADMIN`**
  * `pause()` / `unpause()`
  * `_authorizeUpgrade()` (UUPS)
  * `setGovernance(address)` (one‑time)
  * `setMinUnstakeDelay(uint256)`
* **Governance (external `IORBTGovernance`) via `executeGovernanceAction(type,payload)`**
  * `ACT_SET_RATE` → payload: `uint256 newRateRay` (must be ≥ 1e27)
  * `ACT_SET_REWARD_CONFIG` → payload: `(address rewardToken, address rewardVault, uint256 rewardRatePerSecond)`
  * `ACT_SET_REWARDS_ONLY_MODE` → payload: `bool enabled`

> The reward vault must pre‑fund and `approve` the s0xAsset to `transferFrom` rewards when users claim.

#### ORBT [StakingRewards](../../protocol-mechanics/reward-rate.md#how-orbt-staking-rewards-work)

* **Owner**
  * `setRewardsDuration(uint256)` (can also mid‑epoch: leftover is rescaled)
  * `setRewardsDistribution(address)`
  * `recoverERC20(token, amount)` (cannot recover the staking token)
* **RewardsDistribution**
  * `notifyRewardAmount(uint256 reward)` (must ensure contract balance covers the stream)

***

### Events to index / watch

#### s0xAsset

* `Deposit(address caller, address receiver, uint256 assets, uint256 shares)`
* `Withdraw(address caller, address receiver, address owner, uint256 assets, uint256 shares)`
* `Drip(uint256 newExchangeRateRay, uint256 mintedInterest)`
* `RewardAccrued(uint256 newRewardIndexRay, uint256 rewards, uint256 dt)`
* `RewardClaimed(address user, address to, uint256 amount)`
* `RewardConfigSet(...)`, `RewardsOnlyModeSet(bool)`
* `MinUnstakeDelaySet(uint256 old, uint256 new)`

#### ORBT StakingRewards

* `Staked(address user, uint256 amount)` / `Withdrawn(address user, uint256 amount)`
* `RewardAdded(uint256 reward)` / `RewardPaid(address user, uint256 reward)`
* `RewardsDurationUpdated(uint256 newDuration)` / `RewardsDistributionUpdated(address)`
* `Referral(uint16 referral, address user, uint256 amount)`

***

### Edge cases & gotchas

* **First depositor**: s0xAsset mints `MIN_LIQUIDITY` shares to `address(1)` to avoid rounding issues. The first real depositor gets 1:1 assets → shares at init.
* **Rewards‑only default**: `rewardsOnlyMode = true` after initialization; the interest accumulator is frozen until governance disables this mode. If governance re‑enables rewards‑only later, `rateRay` is snapped to `1e27` internally.
* **Transfer extends lock**: Any inbound share transfer/mint pushes `earliestUnstakeTime[to]` forward by `minUnstakeDelay`.
* **Exit buffer**: `withdraw(assets, ...)` reverts if `assets > totalAssets() * exitBufferBps / 10_000`.
* **Pausable**: User actions and `drip()` respect pause state.
* **Minting requirement**: Underlying must implement `mint(address,uint256)` and grant minter rights to s0xAsset.

***

### Minimal ABI (for light integrators)

{% code overflow="wrap" %}
```solidity
// sOxAsset (subset)
[
  {
    "name": "asset",
    "outputs": [{ "type": "address" }],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "name": "deposit",
    "inputs": [
      { "name": "assets", "type": "uint256" },
      { "name": "receiver", "type": "address" }
    ],
    "outputs": [{ "type": "uint256" }],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "name": "redeem",
    "inputs": [
      { "name": "shares", "type": "uint256" },
      { "name": "receiver", "type": "address" },
      { "name": "owner", "type": "address" }
    ],
    "outputs": [{ "type": "uint256" }],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "name": "convertToAssets",
    "inputs": [{ "name": "shares", "type": "uint256" }],
    "outputs": [{ "type": "uint256" }],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "name": "previewExchangeRateRay",
    "outputs": [{ "type": "uint256" }],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "name": "claim",
    "inputs": [
      { "name": "to", "type": "address" },
      { "name": "amount", "type": "uint256" }
    ],
    "outputs": [{ "type": "uint256" }],
    "stateMutability": "nonpayable",
    "type": "function"
  }
]

// ORBT StakingRewards (subset)
[
  {
    "name": "stake",
    "inputs": [{ "name": "amount", "type": "uint256" }],
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "name": "withdraw",
    "inputs": [{ "name": "amount", "type": "uint256" }],
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "name": "getReward",
    "inputs": [],
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "name": "exit",
    "inputs": [],
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "name": "earned",
    "inputs": [{ "name": "account", "type": "address" }],
    "outputs": [{ "type": "uint256" }],
    "stateMutability": "view",
    "type": "function"
  }
]

```
{% endcode %}

***

### Security notes (for integrators)

* Always **check `preview*`** functions to estimate user outcomes before sending state‑changing txs.
* For s0xAsset:
  * Treat the vault as custodial over `underlyingAsset`; withdrawals can be rate‑limited by `exitBufferBps` and time‑gated by `minUnstakeDelay`.
  * Ensure the **reward vault** is funded and has approved the s0xAsset contract, otherwise `claim` will revert.
  * Upgrades are gated by `ADMIN`; monitor implementation changes.
* For ORBT StakingRewards:
  * `notifyRewardAmount` requires the contract balance to cover the stream; the call reverts if under‑funded.
  * No built‑in slashing/lock - strategies layering locks must be external.

***

### FAQs

<details>

<summary><strong>Q: Do I need to call <code>drip()</code> in my strategy?</strong></summary>

A: Not required. Reads use preview logic. Anyone may call `drip()`; it’s opportunistic.

</details>

<details>

<summary><strong>Q: Can I compound rewards into more shares?</strong></summary>

A: Yes, claim rewards and swap→underlying, then `deposit` again. This compounding can be automated off‑chain.

</details>

<details>

<summary><strong>Q: How is the first deposit priced?</strong></summary>

A: 1:1 assets:shares (minus a tiny `MIN_LIQUIDITY` minted to an unreachable address to stabilize rounding).

</details>

<details>

<summary><strong>Q: Can governance switch to rewards‑only later?</strong></summary>

A: Yes. When enabled, interest stops (rate snaps to 1e27 effectively), but reward emissions can continue.

</details>
