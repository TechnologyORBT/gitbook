# Allocator debt repayment

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
