# Approval management

Recommended pattern:// 1. Before using UPM, approve specific amountWBTC.approve(address(upm), amountNeeded);​// 2. Execute operation via UPMupm.doCall(target, data);​// 3. (Optional) Revoke approval after useWBTC.approve(address(upm), 0);Alternative: Max approval (convenience vs risk):WBTC.approve(address(upm), type(uint256).max);Risk comparison:

|                       |        |            |                                |
| --------------------- | ------ | ---------- | ------------------------------ |
| Exact Amount          | Low    | Cumbersome | One-time users                 |
| Limited (e.g., 1 BTC) | Medium | Balanced   | Regular users                  |
| Max (uint256.max)     | High   | Seamless   | Power users, trusted protocols |
