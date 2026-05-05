# Allocators

Allocators are whitelisted entities (protocols, institutions, DAOs) that receive credit lines to mint 0x assets. They serve as capital-efficiency mechanisms and liquidity providers.

### Allocator Lifecycle

{% stepper %}
{% step %}
### 1. Onboarding (Admin/Governance)

Example call:

```solidity
setAllocatorSingleByAdmin(
  SetAllocatorMemory({ 
    allocator: zeroXAllocatorAddress,
    allowed: true,
    line: LineOfCredit({ ceiling: 1000e18, dailyCap: 100e18, mintedToday: 0, lastMintDay: 0 }),
    borrowFeeBps: 100
  }),
  assets, // [WBTC, cbBTC]
  pockets, // [allocatorWBTCVault, allocatorCbBTCVault]
  AllocatorOperations.SET_ALLOCATOR
);
```
{% endstep %}

{% step %}
### 2. Credit Minting

Example:

```solidity
// Allocator mints 50 0xBTC against credit line
allocatorCreditMint(allocatorAddress, 50e18);
```
{% endstep %}

{% step %}
### 3. Deployment

* Allocator deploys minted 0xBTC into yield strategies
* Allocator earns yield on deployed capital
* Allocator manages risk and liquidity
{% endstep %}

{% step %}
### 4. Repayment

Example:

```solidity
// Allocator returns WBTC to reduce debt
allocatorRepay(WBTC, 50e8); // Repays 50 WBTC worth of debt
```
{% endstep %}
{% endstepper %}

### Allocator Roles

Responsibilities:

* Manage 0x asset inventory responsibly
* Maintain liquidity for referral-attributed swaps
* Repay debt when needed
* Monitor credit utilization and daily caps
* Set custom pockets for underlying asset routing

Privileges:

* Mint 0x assets up to credit line without depositing collateral
* Receive underlying assets from referral swaps directly to their pockets
* Set own pocket addresses via `setMyPocket()`
* Earn yield on deployed capital

Restrictions:

* Cannot redeem 0x assets for underlying
* Must stay within credit ceiling and daily cap
* Subject to borrow fees on minting (if configured)
