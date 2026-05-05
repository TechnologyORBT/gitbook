# Strategy Execution

UPM acts as a controlled execution layer that allows users to perform approved strategy operations through a standardized interface.\
All executable actions must correspond to **whitelisted strategies** registered within the UPM’s internal registry.

This ensures that only verified and approved contracts can be called, maintaining consistency, safety, and protocol-level control.\
No arbitrary or unregistered contract execution is permitted within the UPM framework. &#x20;

## Single Call Execution

The UPM enables execution of a single approved strategy call without deploying new custom contracts.\
Every execution request passes through registry validation to confirm that the target contract is a registered strategy.

Example doCall implementation:

{% code title="UPM.doCall" %}
```solidity
function doCall(address target, bytes memory data) external returns (bytes memory result) {
    result = target.functionCall(data);
}
```
{% endcode %}

#### Use case

&#x20;Simple swap:

```solidity
// Build call data
bytes memory callData = abi.encodeWithSelector(
    OrbtUCE.swapExactIn.selector,
    address(WBTC),
    address(0xBTC),
    1e8,
    msg.sender,
    0
);

// Execute via UPM
upm.doCall(address(orbtUCE), callData);
```

## Batch Call Execution

UPM also supports **batch execution** of multiple approved strategy calls in a single atomic transaction.\
Each target address must exist in the UPM registry; otherwise, the transaction will revert.\
This structure allows for **multi-step strategies** to be executed securely and atomically.

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

#### Use case&#x20;

&#x20;multi-step strategy (atomic):

{% stepper %}
{% step %}
### Swap and deposit to Savings (example)

Build batch calls:

* Step 1: Swap WBTC → 0xBTC via OrbtUCE (UPM receives 0xBTC temporarily)
* Step 2: Deposit 0xBTC to USM, receive s0xBTC (user receives s0xBTC)

Code (conceptual):

```solidity
address[] memory targets = new address[](2);
bytes[] memory datas = new bytes[](2);

// Step 1: Swap
targets[0] = address(orbtUCE);
datas[0] = abi.encodeWithSelector(
    OrbtUCE.swapExactIn.selector,
    address(WBTC),
    address(zeroXBTC),
    1e8,
    address(upm),
    0
);

// Step 2: get s0xAsset
targets[1] = address(sZeroXBTC);
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

{% hint style="success" %}
**Important**: All operations in a batch are atomic - if any step fails, the entire transaction reverts.
{% endhint %}

***

Every strategy executed through the UPM is derived from a **base strategy contract**.\
This base layer standardizes protocol operations and embeds the **fee and** [**buyback model**](../../../protocol-mechanics/orbt-revenue-and-yield-model/immutable-buyback-engine.md), ensuring that a consistent mechanism is followed for yield allocation, treasury contributions, and ecosystem sustainability.

All registered strategies must inherit from this base contract to be eligible for execution within the UPM framework.
