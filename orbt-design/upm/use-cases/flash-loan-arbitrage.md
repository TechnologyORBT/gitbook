# Flash loan arbitrage

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
