# Leveraged staking

{% stepper %}
{% step %}
### Goal

Deposit WBTC → swap to 0xBTC → deposit to savings (s0xBTC) in a single transaction.
{% endstep %}

{% step %}
### Steps (conceptual)

1. Approve UPM to spend user's WBTC (externally)
2. Swap WBTC → 0xBTC (UPM receives 0xBTC temporarily)
3. Approve s0xBTC to spend UPM's 0xBTC
4. Deposit 0xBTC → Savings (user receives s0xBTC)

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
        address(zeroXBTC),
        wbtcAmount,
        address(upm),
        0
    );

    // Step 3: Approve s0xBTC to spend UPM's 0xBTC
    targets[1] = address(zeroXBTC);
    datas[1] = abi.encodeWithSelector(
        IERC20.approve.selector,
        address(sOxBTC),
        type(uint256).max
    );

    // Step 4: Deposit 0xBTC → Savings
    uint256 zeroXAmount = convertToZeroXAssets(wbtcAmount);
    targets[2] = address(sOxBTC);
    datas[2] = abi.encodeWithSelector(
        IERC4626.deposit.selector,
        zeroXAmount,
        msg.sender
    );

    upm.doBatchCalls(targets, datas);
}
```
{% endstep %}
{% endstepper %}
