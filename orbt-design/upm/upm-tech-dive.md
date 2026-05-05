---
hidden: true
---

# UPM   Tech Dive

## Overview

The User Position Manager (UPM) is a flexible smart contract execution layer that enables users to perform complex, multi-step operations with ORBT protocol contracts in a single atomic transaction. Think of it as a programmable router that can batch calls, manage approvals, and execute arbitrary logic on behalf of users—all while maintaining strict security boundaries.

The UPM serves as infrastructure for use cases such as:

* One-click leveraged positions (swap, stake, borrow in one tx)
* Zap contracts (enter complex strategies with one asset)
* Multi-protocol compositions (ORBT + external protocols)
* Gas optimization (batch operations)
* Flash loan integrations (arbitrage, liquidations)

Unlike smart contract wallets, UPM is stateless and permissionless — it does not hold funds between transactions and requires no setup. Frontends and power users compose workflows using low-level primitives (doCall, doBatchCalls) while respecting trust boundaries.

## Architecture

![](<../../.gitbook/assets/Unknown image (1)>)

## Key properties

* Stateless — No user state stored between transactions
* Permissionless — Anyone can call UPM functions
* Non-custodial — Never holds user tokens permanently
* Composable — Works with any external contract
* Gas-efficient — Minimal overhead compared to direct calls

## Concepts

### Arbitrary execution

UPM enables executing arbitrary contract calls without deploying custom contracts.

## Single Call Execution

Example doCall implementation:

{% code title="UPM.doCall" %}
```solidity
function doCall(address target, bytes memory data) external returns (bytes memory result) {
    result = target.functionCall(data);
}
```
{% endcode %}

Use case — Simple swap:

```solidity
// Build call data
bytes memory callData = abi.encodeWithSelector(
    OrbitUCE.swapExactIn.selector,
    address(WBTC),
    address(0xBTC),
    1e8,
    msg.sender,
    0
);

// Execute via UPM
upm.doCall(address(orbitUCE), callData);
```

## Batch Call Execution

Example doBatchCalls implementation:

{% code title="UPM.doBatchCalls" %}
```solidity
function doBatchCalls(
    address[] calldata targets,
    bytes[] calldata datas
) external returns (bytes[] memory results) {
    require(targets.length == datas.length, "UPM/len-mismatch");
    results = new bytes[](targets.length);
    for (uint256 i = 0; i < targets.length; i++) {
        results[i] = targets[i].functionCall(datas[i]);
    }
}
```
{% endcode %}

Use case — multi-step strategy (atomic):

{% stepper %}
{% step %}
### Swap then Stake (example)

Build batch calls:

* Step 1: Swap WBTC → 0xBTC via OrbitUCE (UPM receives 0xBTC temporarily)
* Step 2: Stake 0xBTC to s0xBTC (user receives s0xBTC)

Code (conceptual):

```solidity
address[] memory targets = new address[](2);
bytes[] memory datas = new bytes[](2);

// Step 1: Swap
targets[0] = address(orbitUCE);
datas[0] = abi.encodeWithSelector(
    OrbitUCE.swapExactIn.selector,
    address(WBTC),
    address(0xBTC),
    1e8,
    address(upm),
    0
);

// Step 2: Stake
targets[1] = address(sOxBTC);
datas[1] = abi.encodeWithSelector(
    IERC4626.deposit.selector,
    1e18,
    msg.sender
);

// Execute atomically
upm.doBatchCalls(targets, datas);
```
{% endstep %}
{% endstepper %}

Important: All operations in a batch are atomic — if any step fails, the entire transaction reverts.

## Trust assumptions

What UPM can do:

* Execute calls on your behalf using approvals you've granted
* Transfer your tokens up to approved amounts
* Call any contract (no whitelist)
* Hold tokens temporarily during a batch transaction

What UPM cannot do:

* Steal tokens beyond approvals
* Execute without a user-initiated transaction
* Hold funds permanently (stateless design)
* Change your approvals

## Trust boundary (simplified flow)

User's Wallet

* Holds private keys
* Initiates transactions
* Grants approvals explicitly

\[User grants approval: WBTC → UPM] ▼

User Position Manager (UPM)

* Can spend user's WBTC (up to approved amount)
* Cannot withdraw without user transaction
* Transparent, immutable code

\[UPM executes calls to protocols] ▼

Protocol Contracts (UCE, USM, etc)

* Receive tokens from UPM (via transferFrom)
* Execute business logic
* Return results/tokens to specified recipients

## Security best practices

For users:

* Minimal approvals: Approve only needed amounts
* Revoke unused approvals: Use tools like Revoke.cash
* Verify call data: Ensure batched calls do what you expect
* Use reputable frontends to construct call data
* Check transaction results after execution

For frontend developers:

* Simulate transactions (eth\_call) to preview outcomes
* Validate and sanitize user inputs before encoding
* Show users exact actions the tx will perform
* Provide meaningful error messages
* Include slippage protection in call data

## Approval management

Recommended pattern:

```solidity
// 1. Before using UPM, approve specific amount
WBTC.approve(address(upm), amountNeeded);

// 2. Execute operation via UPM
upm.doCall(target, data);

// 3. (Optional) Revoke approval after use
WBTC.approve(address(upm), 0);
```

Alternative: Max approval (convenience vs risk):

```solidity
WBTC.approve(address(upm), type(uint256).max);
```

Risk comparison:

| Approval Type         | Risk Level | User Experience | Best For                       |
| --------------------- | ---------- | --------------- | ------------------------------ |
| Exact Amount          | Low        | Cumbersome      | One-time users                 |
| Limited (e.g., 1 BTC) | Medium     | Balanced        | Regular users                  |
| Max (uint256.max)     | High       | Seamless        | Power users, trusted protocols |

## Use cases

The multi-step workflows below are large numbered sequences; they are converted into steppers for clarity.

## Leveraged staking (example)

{% stepper %}
{% step %}
### Goal

Deposit WBTC → swap to 0xBTC → stake to s0xBTC in a single transaction.
{% endstep %}

{% step %}
### Steps (conceptual)

1. Approve UPM to spend user's WBTC (externally)
2. Swap WBTC → 0xBTC (UPM receives 0xBTC temporarily)
3. Approve s0xBTC to spend UPM's 0xBTC
4. Stake 0xBTC → s0xBTC (user receives s0xBTC)

Example code skeleton:

```solidity
function leveragedStake(uint256 wbtcAmount) external {
    address[] memory targets = new address[](3);
    bytes[] memory datas = new bytes[](3);

    // Step 2: Swap WBTC → 0xBTC
    targets[0] = address(orbitUCE);
    datas[0] = abi.encodeWithSelector(
        OrbitUCE.swapExactIn.selector,
        address(WBTC),
        address(0xBTC),
        wbtcAmount,
        address(upm),
        0
    );

    // Step 3: Approve s0xBTC to spend UPM's 0xBTC
    targets[1] = address(0xBTC);
    datas[1] = abi.encodeWithSelector(
        IERC20.approve.selector,
        address(sOxBTC),
        type(uint256).max
    );

    // Step 4: Stake 0xBTC → s0xBTC
    uint256 oxAmount = convertToOxAssets(wbtcAmount);
    targets[2] = address(sOxBTC);
    datas[2] = abi.encodeWithSelector(
        IERC4626.deposit.selector,
        oxAmount,
        msg.sender
    );

    upm.doBatchCalls(targets, datas);
}
```
{% endstep %}
{% endstepper %}

## Exit with compound claim

{% stepper %}
{% step %}
### Goal

Unstake s0xBTC, claim rewards, swap rewards to underlying, exit as WBTC.
{% endstep %}

{% step %}
### Steps (conceptual)

1. Redeem s0xBTC → 0xBTC
2. Claim ORBT rewards
3. Swap ORBT → 0xBTC (external DEX)
4. Swap 0xBTC → WBTC (send to user)

Example (conceptual):

```solidity
function exitWithRewards(uint256 shares) external {
    address[] memory targets = new address[](4);
    bytes[] memory datas = new bytes[](4);

    // Step 1: Redeem s0xBTC → 0xBTC
    targets[0] = address(sOxBTC);
    datas[0] = abi.encodeWithSelector(
        IERC4626.redeem.selector,
        shares,
        address(upm),
        msg.sender
    );

    // Step 2: Claim ORBT rewards
    targets[1] = address(sOxBTC);
    datas[1] = abi.encodeWithSelector(
        sOxAsset.claimRewards.selector,
        address(upm)
    );

    // Step 3: Swap ORBT → 0xBTC (dex)
    targets[2] = address(dex);
    datas[2] = abi.encodeWithSelector(
        DEX.swap.selector,
        address(ORBT),
        address(0xBTC),
        orbtBalance,
        0
    );

    // Step 4: Swap 0xBTC → WBTC
    uint256 totalOxBTC = 0xBTC.balanceOf(address(upm));
    targets[3] = address(orbitUCE);
    datas[3] = abi.encodeWithSelector(
        OrbitUCE.swapExactIn.selector,
        address(0xBTC),
        address(WBTC),
        totalOxBTC,
        msg.sender,
        0
    );

    upm.doBatchCalls(targets, datas);
}
```
{% endstep %}
{% endstepper %}

## Flash loan arbitrage

{% stepper %}
{% step %}
### Goal

Borrow from Aave, arbitrage price difference, repay in the same transaction, keep profit.
{% endstep %}

{% step %}
### Steps (conceptual)

1. Initiate flash loan from Aave to UPM
2. Swap WBTC → 0xBTC on ORBT (assume price 1.01)
3. Swap 0xBTC → WBTC on external DEX (assume price 1.00)
4. Repay flash loan + fee (Aave pulls repayment)
5. Keep profit

Concept:

```solidity
function flashArbitrage(uint256 amount) external {
    address[] memory targets = new address[](5);
    bytes[] memory datas = new bytes[](5);

    // Step 1: Initiate flash loan
    targets[0] = address(aave);
    datas[0] = abi.encodeWithSelector(
        Aave.flashLoan.selector,
        address(upm),
        address(WBTC),
        amount,
        // ... flash loan params
    );

    // Steps 2-4 are executed within the flash loan callback
    upm.doBatchCalls(targets, datas);
}
```
{% endstep %}
{% endstepper %}

## Allocator debt repayment

{% stepper %}
{% step %}
### Goal

Allocator repays debt by swapping held assets.
{% endstep %}

{% step %}
### Steps (conceptual)

1. Swap held asset → debt asset via DEX
2. Repay debt to UCE

Example:

```solidity
function repayDebtViaSwap(address assetToSell, uint256 amountToSell, address debtAsset) external {
    address[] memory targets = new address[](2);
    bytes[] memory datas = new bytes[](2);

    // Step 1: Swap asset → debt asset
    targets[0] = address(dex);
    datas[0] = abi.encodeWithSelector(
        DEX.swap.selector,
        assetToSell,
        debtAsset,
        amountToSell,
        0
    );

    // Step 2: Repay debt to UCE
    targets[1] = address(orbitUCE);
    datas[1] = abi.encodeWithSelector(
        OrbitUCE.allocatorRepay.selector,
        debtAsset,
        debtAssetAmount
    );

    upm.doBatchCalls(targets, datas);
}
```
{% endstep %}
{% endstepper %}

## Integration guide

### For frontend developers

Basic integration:

```javascript
// 1. Import UPM contract
import { UPM_ADDRESS, UPM_ABI } from '@orbt/contracts';

// 2. Encode call data
const callData = orbitUCE.interface.encodeFunctionData('swapExactIn', [
  WBTC_ADDRESS,
  OXBTC_ADDRESS,
  amountIn,
  userAddress,
  0
]);

// 3. Execute via UPM
const tx = await upm.doCall(ORBITUCE_ADDRESS, callData);
await tx.wait();
```

Batch calls:

```javascript
const targets = [ORBITUCE_ADDRESS, S0XBTC_ADDRESS];
const datas = [
  orbitUCE.interface.encodeFunctionData('swapExactIn', [...]),
  sOxBTC.interface.encodeFunctionData('deposit', [...])
];

const tx = await upm.doBatchCalls(targets, datas);
await tx.wait();
```

Error handling:

```javascript
try {
  const tx = await upm.doBatchCalls(targets, datas);
  await tx.wait();
  console.log('Success!');
} catch (error) {
  if (error.message.includes('UPM/len-mismatch')) {
    console.error('Targets and datas length mismatch');
  } else if (error.message.includes('revert')) {
    console.error('One of the calls reverted:', error);
  } else {
    console.error('Unknown error:', error);
  }
}
```

### For smart contract developers

Calling UPM from another contract:

```solidity
contract MyStrategy {
    OrbitUPM public immutable upm;
    constructor(address upm_) { upm = OrbitUPM(upm_); }

    function executeComplexStrategy() external {
        address[] memory targets = new address[](2);
        bytes[] memory datas = new bytes[](2);
        // populate...
        bytes[] memory results = upm.doBatchCalls(targets, datas);
        uint256 outputAmount = abi.decode(results[0], (uint256));
    }
}
```

Receiving tokens from UPM:

```solidity
contract MyVault {
    function deposit(uint256 amount) external {
        // UPM can call this after swapping tokens to this contract
        IERC20(token).transferFrom(msg.sender, address(this), amount);
    }
}
```

## Security considerations

Audit status

* Status: \[Pending/Completed]
* Auditor: \[Firm Name]
* Report: \[Link to audit report]

Known risks

1. Approval risk — granting max approval to UPM is a trust decision
2. Call data risk — malicious call data could drain tokens if frontend is untrusted
3. Reentrancy — UPM itself has no reentrancy guards; targets must be safe
4. Flash loan attacks — UPM can be used to facilitate attacks elsewhere
5. Gas limits — very large batches may hit block gas limits

Mitigation strategies

* Protocol level: minimal audited code, use Address.functionCall (OpenZeppelin), immutable deployment
* User level: use reputable frontends, review txs, manage approvals
* Frontend level: simulate txs, preview actions, slippage protection, validate inputs

Emergency response

1. Community alert (Discord/Twitter/forum)
2. Revoke approvals
3. Deploy patched UPM (new address)
4. Coordinate migration

## Gas optimization

* Single call overhead: \~+3,000 gas vs direct (small)
* Batch calls: batching N calls reduces overhead compared to N separate txs; break-even typically N ≥ 3
* Tips:
  * Batch when possible (3+)
  * Minimize calldata size
  * Reuse approvals
  * estimateGas before sending

Comparison with alternatives

| Feature         | UPM           | Smart Wallet | Multicall     | Custom Router |
| --------------- | ------------- | ------------ | ------------- | ------------- |
| Permissionless  | ✅             | ❌            | ✅             | Varies        |
| Stateless       | ✅             | ❌            | ✅             | Varies        |
| Arbitrary calls | ✅             | ✅            | ✅             | ❌             |
| Gas efficiency  | High          | Low          | High          | Highest       |
| Setup required  | No            | Yes          | No            | N/A           |
| Upgradeable     | No            | Yes          | No            | Varies        |
| Custody         | Non-custodial | Self-custody | Non-custodial | Non-custodial |

## FAQ (expandable)

<details>

<summary>Q: Do I need to deploy my own UPM?</summary>

A: No, UPM is a shared, permissionless contract. Everyone can use the same deployment.

</details>

<details>

<summary>Q: Can UPM steal my tokens?</summary>

A: Only if you've approved it to spend them. UPM operates within approval boundaries.

</details>

<details>

<summary>Q: What happens if a batch call fails?</summary>

A: The entire transaction reverts; no partial execution occurs.

</details>

<details>

<summary>Q: Can I use UPM with other protocols (Uniswap, Aave, etc.)?</summary>

A: Yes — UPM can call any contract.

</details>

<details>

<summary>Q: Is UPM upgradeable?</summary>

A: No — UPM deployments are immutable. Fixes require a new deployment and migration.

</details>

<details>

<summary>Q: Can I cancel a UPM transaction after sending?</summary>

A: No — once the transaction is confirmed on-chain it executes. Review carefully before signing.

</details>

## Appendix: Code snippets, examples, and reference

(Full example code, integration snippets, and earlier function implementations are preserved above in their respective sections.)

## Deployment info

(Provide network and deployment details where appropriate in your repo or frontend; not included here.)

***

If you want, I can:

* Extract specific code examples into separate files (e.g., Solidity sample strategies).
* Produce a short frontend checklist for integrating UPM safely.
* Create a one-page quickstart for users (approve → build calls → execute → optional revoke). Which would you prefer?
