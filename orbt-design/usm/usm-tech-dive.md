---
hidden: true
---

# USM   Tech Dive

User Staking Module (USM) Overview

The User Staking Module (USM) provides yield-bearing staking for 0x assets through the sOxAsset contracts (e.g., s0xBTC, s0xETH, s0xUSD). Built on the ERC-4626 tokenized vault standard, the USM enables users to deposit 0x assets and receive interest-bearing shares that appreciate in value over time through a governance-controlled interest rate mechanism.

The USM implements an accumulator-based savings model inspired by MakerDAO's DSR/sUSDS, where a global exchange rate grows continuously based on a per-second interest rate. Key advantages:

* Gas Efficiency: interest accrual is computed algorithmically without per-user updates
* Composability: full ERC-4626 compatibility integrates with DeFi protocols
* Dual Yield Mechanism: native interest accrual + optional external reward token emissions
* Governance Control: interest rates adjustable via multisig/timelock governance
* Flexible Exit: configurable withdrawal limits and unstake delays
* Rewards-Only Mode: opt to accrue only external rewards (no minting of underlying)

s0xAsset tokens are ERC-20 claims on a growing pool of underlying 0x assets. Users can enter/exit (subject to delays) and shares appreciate via the global interest rate.

Architecture

{% stepper %}
{% step %}
### Core Vault Layer (ERC-4626)

![](<../../.gitbook/assets/unknown (1).png>)
{% endstep %}

{% step %}
### Governance Integration Layer

The USM integrates with the ORBT governance system for secure rate management.

Supported governance actions:

* SET\_RATE: Update the per-second interest rate
* SET\_REWARD\_CONFIG: Configure external reward token emissions
* SET\_REWARDS\_ONLY\_MODE: Toggle between full accrual and rewards-only mode

Example governance interface:

```solidity
interface IGovernedContract {
    function executeGovernanceAction(bytes32 actionType, bytes calldata payload) external returns (bool);
}
```
{% endstep %}

{% step %}
### User Interface Layer

Users interact through two primary flows (Staking / Unstaking). See the Staking Flow and Unstaking Flow stepper below.
{% endstep %}
{% endstepper %}

User flows

{% stepper %}
{% step %}
### Staking Flow

User → approve(0xBTC) → deposit(0xBTC) → Mint s0xBTC shares → Hold and accrue interest
{% endstep %}

{% step %}
### Unstaking Flow

User → redeem(s0xBTC) → Burn shares → Receive 0xBTC principal + interest

(Withdraws subject to exit buffer and unstake delay config.)
{% endstep %}
{% endstepper %}

Core concepts

ERC-4626 Tokenized Vault Standard

USM implements the full ERC-4626 spec. Core functions (examples):

Deposit functions

```solidity
// Deposit exact assets, receive shares
function deposit(uint256 assets, address receiver) external returns (uint256 shares);

// Mint exact shares, deposit required assets
function mint(uint256 shares, address receiver) external returns (uint256 assets);
```

Withdrawal functions

```solidity
// Withdraw exact assets, burn required shares
function withdraw(uint256 assets, address receiver, address owner) external returns (uint256 shares);

// Redeem exact shares, receive assets
function redeem(uint256 shares, address receiver, address owner) external returns (uint256 assets);
```

Preview functions

```solidity
function previewDeposit(uint256 assets) external view returns (uint256 shares);
function previewMint(uint256 shares) external view returns (uint256 assets);
function previewWithdraw(uint256 assets) external view returns (uint256 shares);
function previewRedeem(uint256 shares) external view returns (uint256 assets);
```

Accounting / compliance benefits

* Standardized interface: works with ERC-4626-compatible protocols
* Composability: can be used as collateral, in yield aggregators
* Tooling support: compatible with vault management tools
* Auditor familiarity, future-proof integrations

Interest accrual model

Accumulator-based model where interest compounds continuously.

Key constants / state (RAY = 1e27 fixed-point):

```solidity
uint256 internal constant RAY = 1e27;
uint256 public exchangeRateRay; // assets per 1 share (init = RAY)
uint256 public rateRay;         // per-second interest factor (RAY)
uint256 public lastAccrual;     // timestamp
```

Drip mechanism

The drip() function updates exchangeRateRay based on elapsed time:

```solidity
function drip() public {
    uint256 currentTime = block.timestamp;
    uint256 elapsed = currentTime - lastAccrual;
    if (elapsed == 0) return;

    // compound: exchangeRate' = exchangeRate * (rateRay ^ elapsed)
    uint256 compoundFactor = _rpow(rateRay, elapsed);
    uint256 newExchangeRateRay = (exchangeRateRay * compoundFactor) / RAY;

    uint256 totalSharesBefore = totalSupply();
    if (totalSharesBefore > 0 && !rewardsOnlyMode) {
        uint256 assetsBefore = (totalSharesBefore * exchangeRateRay) / RAY;
        uint256 assetsAfter  = (totalSharesBefore * newExchangeRateRay) / RAY;
        uint256 interestAccrued = assetsAfter - assetsBefore;

        // Mint new underlying to vault
        IMintableERC20(address(underlyingAsset)).mint(address(this), interestAccrued);

        emit Drip(newExchangeRateRay, interestAccrued);
    }

    exchangeRateRay = newExchangeRateRay;
    lastAccrual = currentTime;
}
```

Mathematical foundation

A(t) = P \* (1 + r)^t

In RAY precision: e.g., 3.2% APY → per-second rate \~1.000000001 RAY; after 1 year ≈ 1.032 RAY.

Rate conversion example (APY to per-second):

```solidity
// rateRay = (1 + APY)^(1/secondsPerYear) * RAY
// e.g., 5% APY -> rateRay ≈ 1.000000001547125 RAY
```

Yield accrual mechanics

1. Native yield — exchange rate appreciation

* Shares represent claim on growing assets; deposits convert assets→shares using exchangeRateRay.

2. External reward tokens

* rewardToken (IERC20), rewardVault (pre-funded), rewardRatePerSecond
* Per-user accounting:

```solidity
mapping(address => uint256) public userRewardIndexRay;
mapping(address => uint256) public accruedRewards;
uint256 public rewardIndexRay;

function _updateUserRewards(address user) internal {
  uint256 userShares = balanceOf(user);
  if (userShares == 0) return;
  uint256 indexDelta = rewardIndexRay - userRewardIndexRay[user];
  uint256 rewardsEarned = (userShares * indexDelta) / RAY;
  accruedRewards[user] += rewardsEarned;
  userRewardIndexRay[user] = rewardIndexRay;
}
```

Claiming rewards

```solidity
function claimRewards(address to) external returns (uint256 claimed) {
  _accrueRewards();
  _updateUserRewards(msg.sender);
  claimed = accruedRewards[msg.sender];
  if (claimed > 0) {
    accruedRewards[msg.sender] = 0;
    IERC20(rewardToken).safeTransferFrom(rewardVault, to, claimed);
    emit RewardClaimed(msg.sender, to, claimed);
  }
}
```

Reward configuration (governance)

```solidity
function executeGovernanceAction(bytes32 actionType, bytes calldata payload) external {
  if (actionType == ACT_SET_REWARD_CONFIG) {
    (address token, address vault, uint256 rate) = abi.decode(payload, (address, address, uint256));
    _setRewardConfig(token, vault, rate);
  }
}
```

_rewardsOnlyMode_ — drip() always accrues external rewards; minting underlying interest is skipped when enabled.

Exit buffer and withdraw limits

* exitBufferBps: max withdrawable % of total assets (e.g., 5000 = 50%)
* maxWithdraw(owner) enforces per-user and protocol limits + unstake delay.

Unstake delay

* minUnstakeDelay (seconds)
* earliestUnstakeTime mapping
* Transfers/deposits update earliestUnstakeTime for recipients.
* Withdraw checks require timestamp >= earliestUnstakeTime\[owner].

Governance integration

Governance actions are executed via executeGovernanceAction; examples:

* SET\_RATE: drip(); set new rateRay; emit RateSet
* SET\_REWARD\_CONFIG: \_accrueRewards(); set rewardToken, vault, rate
* SET\_REWARDS\_ONLY\_MODE: drip(); set rewardsOnlyMode

Governance example execution flow

```solidity
function executeGovernanceAction(bytes32 actionType, bytes calldata payload) external onlyGovernance returns (bool) {
  if (actionType == ACT_SET_RATE) {
    uint256 newRate = abi.decode(payload, (uint256));
    _handleSetRate(newRate);
    return true;
  } else if (actionType == ACT_SET_REWARD_CONFIG) {
    (address token, address vault, uint256 rate) = abi.decode(payload, (address, address, uint256));
    _handleSetRewardConfig(token, vault, rate);
    return true;
  } else if (actionType == ACT_SET_REWARDS_ONLY_MODE) {
    bool enabled = abi.decode(payload, (bool));
    _handleSetRewardsOnlyMode(enabled);
    return true;
  }
  revert("Unknown action");
}
```

Integration examples

For users — basic staking

```solidity
// 1. Approve USM
0xBTC.approve(address(s0xBTC), 100e18);

// 2. Deposit
uint256 shares = s0xBTC.deposit(100e18, msg.sender);

// 3. Wait and accrue yield...

// 4. Redeem (after unstake delay if applicable)
uint256 assets = s0xBTC.redeem(shares, msg.sender, msg.sender);
```

With external rewards

```solidity
// After holding for 1 week
s0xBTC.claimRewards(msg.sender); // receives ORBT rewards
```

Yield aggregation example

```solidity
function harvest() external {
  // 1. Claim rewards
  uint256 rewards = s0xBTC.claimRewards(address(this));

  // 2. Swap rewards for underlying

  // 3. Re-stake
  s0xBTC.deposit(underlying, address(this));
}
```

Security considerations

1. Minter role: underlying must grant MINTER to s0xAsset for interest minting
2. Rate limits: governance validates rates to avoid overflow
3. Reward vault solvency: monitor & refill as needed
4. Flash-loan protection: unstake delay prevents same-block exploits
5. ERC-4626 compliance: follow spec exactly
6. Rounding: favor protocol (round down on withdrawals, up on deposits)
7. Exchange rate precision: use RAY (27 decimals) to minimize rounding errors

Gas optimization tips

* Call drip() before rate-sensitive operations
* Batch reward claims
* Use previewRedeem off-chain
* Consider delegating reward claims to reduce user gas

FAQ

<details>

<summary>Q: Can I withdraw immediately after depositing?</summary>

A: Depends on minUnstakeDelay. If set to 0, yes. Otherwise, you must wait.

</details>

<details>

<summary>Q: Do I need to claim rewards to earn them?</summary>

A: No, rewards accrue automatically. Claiming just transfers them to your wallet.

</details>

<details>

<summary>Q: What happens if the reward vault runs out of tokens?</summary>

A: Reward claims will fail. Monitor vault balance and refill as needed.

</details>

<details>

<summary>Q: Can the exchange rate decrease?</summary>

A: No, rateRay is always ≥ 1 RAY. The exchange rate can only increase or stay flat.

</details>

<details>

<summary>Q: Is s0xBTC transferable?</summary>

A: Yes, fully transferable like any ERC-20. Receiving tokens may reset your unstake timer.

</details>

Appendix / Examples & Snippets

* Exchange rate accumulator (RAY precision) and drip() example shown above
* Reward vault setup example (fund vault, approve contract, set reward rate via governance) — keep vault funded and allowances set
* APY→per-second examples (see Rate Conversion Examples section above)

Notes

* The repository examples and code snippets are illustrative; maintain exact types, decimals and edge-case checks (overflows, zero cases) in production.
* The base64 diagram in the Architecture section is kept intact.
