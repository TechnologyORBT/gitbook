# Setting up Allocators

Full allocator setup (single action example):

```solidity
SetAllocatorMemory memory config = SetAllocatorMemory({
    allocator: 0xAllocatorAddress,
    allowed: true,
    line: LineOfCredit({
        ceiling: 5000e18,
        dailyCap: 500e18,
        mintedToday: 0,
        lastMintDay: 0
    }),
    borrowFeeBps: 50
});

address[] memory assets = new address[](2);
assets[0] = address(WBTC);
assets[1] = address(cbBTC);

address[] memory pockets = new address[](2);
pockets[0] = 0xAllocatorWBTCVault;
pockets[1] = 0xAllocatorCbBTCVault;

// Execute via governance:
bytes memory payload = abi.encode(AllocatorOperations.SET_ALLOCATOR, config, assets, pockets);
```

Partial updates (granular control):

* Update allowed status only: AllocatorOperations.UPDATE\_ALLOWED
* Update credit line only: AllocatorOperations.UPDATE\_LINE
* Update borrow fee only: AllocatorOperations.UPDATE\_BORROW\_FEE
* Update pockets (via governance): payload: (allocator, asset, newPocket) for SET\_ALLOCATOR\_POCKETS

Admin quick actions (non-governance):

```solidity
setAllocatorSingleByAdmin(config, assets, pockets, AllocatorOperations.UPDATE_ALLOWED);
setAllocatorSingleByAdmin(config, assets, pockets, AllocatorOperations.UPDATE_LINE);
```
