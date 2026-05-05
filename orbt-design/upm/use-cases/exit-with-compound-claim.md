# Exit with compound claim

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
