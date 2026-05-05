# For smart contract developers

Calling UPM from another contract:

```solidity
contract MyStrategy {
    OrbtUPM public immutable upm;
    constructor(address upm_) { upm = OrbtUPM(upm_); }

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
