# Arbitrary Execution

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
