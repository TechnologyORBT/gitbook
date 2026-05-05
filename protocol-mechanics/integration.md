# Integration

## Overview

OrbitUCE enables deterministic swaps between underlying assets and their synthetic 0x-wrapped equivalents, as well as between 0x assets and their staked ERC-4626 vault shares (sOx assets). All swaps are executed at par after decimal normalization, with no AMM-style pricing curves.

Key Features:

* 1:1 exchange rate (decimal-normalized)
* No price slippage or spread
* Deterministic quotes via preview functions
* Optional referral-based routing
* Support for ERC-4626 vault integrations

***

## Supported Swap Routes

Supported Routes:

* Underlying ↔ 0x (Bidirectional): WBTC ↔ oxBTC, USDC/USDT ↔ oxUSD
* 0x ↔ sOx (Bidirectional): oxBTC ↔ soxBTC (ERC-4626 shares)

Not Supported:

* Underlying ↔ Underlying: Direct swaps between different underlyings
* Underlying ↔ sOx: Must route through 0x asset

***

## Core Functions

{% stepper %}
{% step %}
### Exact Input Swap

Execute a swap with a known input amount.

Function signature:

{% code title="swapExactIn" %}
```solidity
function swapExactIn(
  address assetIn,
  address assetOut,
  uint256 amountIn,
  address receiver,
  uint256 referralCode
) external nonReentrant whenNotPaused returns (uint256 amountOut);
```
{% endcode %}

Parameters:

* assetIn — Address of the input token
* assetOut — Address of the output token
* amountIn — Exact amount of input token to swap
* receiver — Address that receives the output tokens (must be non-zero)
* referralCode — Optional routing identifier; use 0 if none

Returns:

* amountOut — Actual amount of output tokens delivered

Gas estimate: \~120k–180k (varies by route and liquidity source)
{% endstep %}

{% step %}
### Exact Output Swap

Execute a swap targeting a specific output amount.

Function signature:

{% code title="swapExactOut" %}
```solidity
function swapExactOut(
  address assetIn,
  address assetOut,
  uint256 amountOut,
  uint256 maxAmountIn,
  address receiver,
  uint256 referralCode
) external nonReentrant whenNotPaused returns (uint256 amountIn);
```
{% endcode %}

Parameters:

* assetIn — Address of the input token
* assetOut — Address of the output token
* amountOut — Exact amount of output token desired
* maxAmountIn — Maximum input tokens willing to spend (slippage protection)
* receiver — Address that receives the output tokens (must be non-zero)
* referralCode — Optional routing identifier; use 0 if none

Returns:

* amountIn — Actual amount of input tokens consumed

Reverts if: amountIn > maxAmountIn
{% endstep %}
{% endstepper %}

***

## Preview Functions

Always use preview functions before executing swaps to get deterministic quotes.

### Preview Exact Input

Function signature:

{% code title="previewSwapExactIn" %}
```solidity
function previewSwapExactIn(
  address assetIn,
  address assetOut,
  uint256 amountIn
) external view returns (uint256 amountOut);
```
{% endcode %}

### Preview Exact Output

Function signature:

{% code title="previewSwapExactOut" %}
```solidity
function previewSwapExactOut(
  address assetIn,
  address assetOut,
  uint256 amountOut
) external view returns (uint256 amountIn);
```
{% endcode %}

### Decimal Conversion Helpers

Function signatures:

{% code title="convertToOxAssets / convertToAssets" %}
```solidity
function convertToOxAssets(address asset, uint256 assets)
  external view returns (uint256 oxAmount);

function convertToAssets(address asset, uint256 oxAmount)
  external view returns (uint256 assets);
```
{% endcode %}

Use case: Converting between assets with different decimals (e.g., USDC 6 decimals → oxUSD 18 decimals).

***

## Token Approvals

Before executing swaps, set appropriate token allowances.

Required approvals by swap direction:

* Underlying → 0x: IERC20(assetIn).approve(OrbitUCE, amountIn)
* 0x → Underlying: None (contract burns from msg.sender)
* 0x → sOx: IERC20(oxAsset).approve(OrbitUCE, amountIn)
* sOx → 0x: IERC20(sOxAsset).approve(OrbitUCE, amountIn)

Best practice: Set allowances to exact swap amounts rather than infinite approvals.

***

## Referral System

### Overview

Referral codes enable routing swaps through specific allocator liquidity pockets. When a valid referral code is provided:

* Underlying → 0x: User's underlying is routed to the allocator's pocket; allocator's reserved 0x inventory is used for output
* 0x output paths: Allocator's reserved inventory is consumed first

### Usage

Query allocator for a referral code:

```solidity
address allocator = orbitUCE.referralToAllocator(referralCode);
```

Use in swap (pass 0 for default routing):

```solidity
swapExactIn(assetIn, assetOut, amountIn, receiver, referralCode);
```

Important: Using a referral code does NOT reduce the allocator's debt. Debt reduction only occurs via explicit allocatorRepay() calls.

***

## Exchange Rate Mechanics

### Base Rate

All swaps execute at 1:1 par after decimal normalization:

* 1 WBTC (8 decimals) = 10^10 oxBTC (18 decimals)
* 1 USDC (6 decimals) = 10^12 oxUSD (18 decimals)

### Dynamic Redemption Rate (0x → Underlying Only)

When redeeming 0x assets for underlying, a dynamic fee affects internal accounting:

* Base Rate: Starts at 0%, increases with redemption volume
* Decay: \~0.5% per hour
* Maximum: 5%
* Minimum: 0%

Critical: This fee affects internal debt accounting but does NOT reduce the user's output amount. User swaps remain 1:1.

Query current rate:

```solidity
uint256 currentRate = orbitUCE.getCurrentRedemptionRate(); // 1e18 precision
```

***

## Error Handling

### Common Revert Reasons

Asset Validation:

* OrbitUCE\_\_InvalidAsset — Asset not supported
* OrbitUCE\_\_InvalidAssetIn — Input asset invalid
* OrbitUCE\_\_InvalidAssetOut — Output asset invalid
* OrbitUCE\_\_AssetIsPaused — Asset temporarily paused
* OrbitUCE\_\_UnsupportedSwap — Invalid swap pair

Input Validation:

* OrbitUCE\_\_InvalidAmount — Zero or invalid input amount
* OrbitUCE\_\_InvalidAmountOut — Zero output amount
* OrbitUCE\_\_InvalidReceiver — Receiver is zero address
* OrbitUCE\_\_AmountInTooHigh — Exceeded maxAmountIn slippage protection

Allocator Restrictions:

* OrbitUCE\_\_AllocatorMustSwapUnderlying — Allocator used wrong function
* OrbitUCE\_\_InsufficientAllocatorBalance — Allocator inventory depleted
* OrbitUCE\_\_InsufficientAllocatorPocketLiquidity — Pocket liquidity insufficient

Settlement:

* OrbitUCE\_\_InsufficientPocketLiquidity — Global pocket liquidity depleted
* OrbitUCE\_\_SettlementMismatch — ERC-4626 rounding mismatch

### Pause States

Check pause status before executing swaps:

```solidity
// Contract-level pause
bool isContractPaused = orbitUCE.paused();
// Asset-level pause
bool isAssetPaused = orbitUCE.assetPaused(asset);
```

***

## Allocator Restrictions

Critical for market makers and professional users:

* If your address is registered as an allocator:
  * Cannot execute swapExactIn/Out for 0x → Underlying direction
  * Must use allocatorRepay(address asset, uint256 assets) instead
  * Can execute all other swap directions normally

Check allocator status:

```solidity
bool isAllocator = orbitUCE.isAllocator(yourAddress);
```

***

## Integration Examples

{% stepper %}
{% step %}
#### Example: Underlying → 0x (Exact Input)

```solidity
// 1. Get 0x asset address
address oxAsset = orbitUCE.oxAsset();
// 2. Preview swap
uint256 expectedOut = orbitUCE.previewSwapExactIn(
  wbtc,
  oxAsset,
  1e8 // 1 WBTC
);
// 3. Check asset status
require(!orbitUCE.assetPaused(wbtc), "Asset paused");
require(!orbitUCE.paused(), "Contract paused");
// 4. Approve tokens
IERC20(wbtc).approve(address(orbitUCE), 1e8);
// 5. Execute swap
uint256 actualOut = orbitUCE.swapExactIn(
  wbtc,      // assetIn
  oxAsset,   // assetOut
  1e8,       // amountIn
  msg.sender,// receiver
  0          // referralCode
);
```
{% endstep %}

{% step %}
#### Example: 0x → Underlying (Exact Output)

```solidity
// 1. Preview required input
uint256 requiredIn = orbitUCE.previewSwapExactOut(
  oxAsset,
  usdc,
  1000e6 // Want exactly 1000 USDC
);
// 2. Set slippage tolerance (0.1%)
uint256 maxAmountIn = requiredIn * 1001 / 1000;
// 3. Execute swap (no approval needed - burns from sender)
uint256 actualIn = orbitUCE.swapExactOut(
  oxAsset,   // assetIn
  usdc,      // assetOut
  1000e6,    // amountOut
  maxAmountIn,// maxAmountIn
  msg.sender,// receiver
  0          // referralCode
);
```
{% endstep %}

{% step %}
#### Example: 0x → sOx (Exact Input)

```solidity
// 1. Get vault address for sOx asset
address sOxVault = /* from protocol registry */;
// 2. Preview shares output
uint256 expectedShares = orbitUCE.previewSwapExactIn(
  oxAsset,
  sOxVault,
  100e18 // 100 oxAsset
);
// 3. Approve 0x tokens
IERC20(oxAsset).approve(address(orbitUCE), 100e18);
// 4. Execute swap
uint256 actualShares = orbitUCE.swapExactIn(
  oxAsset,
  sOxVault,
  100e18,
  msg.sender,
  0
);
```
{% endstep %}

{% step %}
#### Example: sOx → 0x (Exact Output with ERC-4626)

```solidity
// 1. Preview required shares
uint256 requiredShares = orbitUCE.previewSwapExactOut(
  sOxVault,
  oxAsset,
  50e18 // Want exactly 50 oxAsset
);
// 2. Approve shares
IERC20(sOxVault).approve(address(orbitUCE), requiredShares);
// 3. Execute with tight slippage
uint256 actualShares = orbitUCE.swapExactOut(
  sOxVault,
  oxAsset,
  50e18,
  requiredShares, // Exact amount from preview
  msg.sender,
  0
);
```
{% endstep %}
{% endstepper %}

***

## JavaScript / TypeScript Integration

### ethers.js v6 Example

```javascript
import { ethers } from 'ethers';
const uce = new ethers.Contract(UCE_ADDRESS, OrbitUCE_ABI, signer);
// Get 0x asset address
const oxAddress = await uce.oxAsset();
// Preview swap: 1000 USDC → oxUSD
const amountIn = ethers.parseUnits("1000", 6);
const expectedOut = await uce.previewSwapExactIn(USDC, oxAddress, amountIn);
// Approve USDC
const usdcContract = new ethers.Contract(USDC, ERC20_ABI, signer);
await usdcContract.approve(UCE_ADDRESS, amountIn);
// Execute swap
const tx = await uce.swapExactIn(
  USDC,
  oxAddress,
  amountIn,
  await signer.getAddress(),
  0 // no referral
);
await tx.wait();
```

### viem Example

```javascript
import { createPublicClient, createWalletClient, http } from 'viem';
const publicClient = createPublicClient({
  transport: http()
});
// Read 0x asset address
const oxAddress = await publicClient.readContract({
  address: UCE_ADDRESS,
  abi: OrbitUCE_ABI,
  functionName: 'oxAsset'
});
// Preview swap
const expectedOut = await publicClient.readContract({
  address: UCE_ADDRESS,
  abi: OrbitUCE_ABI,
  functionName: 'previewSwapExactIn',
  args: [USDC, oxAddress, amountIn]
});
// Approve
await walletClient.writeContract({
  address: USDC,
  abi: ERC20_ABI,
  functionName: 'approve',
  args: [UCE_ADDRESS, amountIn]
});
// Execute swap
await walletClient.writeContract({
  address: UCE_ADDRESS,
  abi: OrbitUCE_ABI,
  functionName: 'swapExactIn',
  args: [USDC, oxAddress, amountIn, receiverAddress, 0n]
});
```

***

## Events

Monitor swap execution via the Swap event.

Event signature:

```solidity
event Swap(
  address indexed assetIn,
  address indexed assetOut,
  address indexed sender,
  address receiver,
  uint256 amountIn,
  uint256 amountOut,
  uint256 referralCode
);
```

Indexing strategy:

* Index by sender for user activity tracking
* Index by referralCode for attribution analytics
* Track amountIn/amountOut for volume metrics

***

## Best Practices

1.  Always preview before executing

    ```solidity
    uint256 expected = orbitUCE.previewSwapExactIn(assetIn, assetOut, amount);
    uint256 actual = orbitUCE.swapExactIn(assetIn, assetOut, amount, receiver, 0);
    ```

    Avoid calling swap without preview.
2.  Handle pauses gracefully

    ```solidity
    function safeSwap() external {
      if (orbitUCE.paused()) {
        revert("Service temporarily unavailable");
      }
      if (orbitUCE.assetPaused(assetIn)) {
        revert("Asset temporarily unavailable");
      }
      // Execute swap
    }
    ```
3.  Use tight allowances Preferred:

    ```solidity
    IERC20(token).approve(uce, amountNeeded);
    ```

    Acceptable (with approval management):

    ```solidity
    IERC20(token).approve(uce, type(uint256).max);
    ```
4.  Validate referral codes

    ```solidity
    function swapWithReferral(uint256 code) external {
      address allocator = orbitUCE.referralToAllocator(code);
      if (allocator == address(0)) {
        // Invalid code - use default routing
        code = 0;
      }
      orbitUCE.swapExactIn(assetIn, assetOut, amount, receiver, code);
    }
    ```
5. ERC-4626 rounding considerations
   *   For sOx swaps, prefer swapExactOut with preview to avoid SettlementMismatch:

       ```solidity
       uint256 sharesNeeded = previewSwapExactOut(sOx, ox, targetAssets);
       swapExactOut(sOx, ox, targetAssets, sharesNeeded, receiver, 0);
       ```
   *   Alternatively, exact-in may have tiny rounding differences:

       ```solidity
       uint256 assetsOut = previewSwapExactIn(sOx, ox, shares);
       swapExactIn(sOx, ox, shares, receiver, 0);
       ```

***

## Liquidity Considerations

Underlying → 0x:

* Always succeeds (mints new 0x tokens as needed)
* No liquidity check required
* Consistent gas costs

0x → Underlying:

* Requires sufficient underlying in pockets
* Check pocket balance before large swaps
* May revert with InsufficientPocketLiquidity

Checking available liquidity:

```solidity
address pocket = orbitUCE.pockets(underlyingAsset);
uint256 pocketBalance = IERC20(underlyingAsset).balanceOf(pocket);
uint256 allowance = IERC20(underlyingAsset).allowance(pocket, address(orbitUCE));
uint256 availableLiquidity = min(pocketBalance, allowance);
```

***

## Gas Optimization

Function selection:

* swapExactIn: \~120k–150k gas (typical)
  * Cheaper when you know input amount
  * Single decimal conversion
* swapExactOut: \~130k–180k gas (typical)
  * Required when targeting exact output
  * Additional validation logic

Batch operations:

* Do not batch reentrant-sensitive calls in the same transaction (contract uses ReentrancyGuard).
* Submit separate transactions or use multicall at router level (non-reentrant calls).

Approval optimization:

```solidity
// Check existing allowance first
uint256 currentAllowance = IERC20(token).allowance(user, uce);
if (currentAllowance < amountNeeded) {
  IERC20(token).approve(uce, amountNeeded);
}
```

***

## Security Considerations

1. Reentrancy protection
   * All swap functions use nonReentrant modifier (OpenZeppelin ReentrancyGuard).
2. Safe token transfers
   * Uses OpenZeppelin's SafeERC20 for transfers and handles non-standard ERC20 implementations.
3.  Slippage protection

    ```solidity
    // For swapExactOut, always set reasonable maxAmountIn
    uint256 maxIn = previewAmount * 1001 / 1000; // 0.1% tolerance
    swapExactOut(assetIn, assetOut, amountOut, maxIn, receiver, 0);
    ```
4. Front-running mitigation
   * 1:1 exchange rate eliminates sandwich attack profitability.
   * Dynamic redemption rate affects only internal accounting.
   * No MEV extraction opportunity on user swaps.
5. Input validation
   * All functions validate non-zero amounts, non-zero receiver addresses, supported assets, and valid swap routes.

***

## Integration Checklist

* Query 0x asset address via oxAsset()
* Verify assets are supported (preview won't revert)
* Implement preview before all swaps
* Set appropriate token allowances
* Handle pause states gracefully
* Validate receiver addresses (non-zero)
* Implement referral system (or default to 0)
* Handle all documented error cases
* For sOx swaps, prefer exact-out with previews
* Check allocator status if relevant
* Monitor Swap events for confirmation
* Test with small amounts on testnet first

***
