# Overview

The User Position Manager (UPM) is a flexible smart contract execution layer that enables users to perform complex, multi-step operations with ORBT protocol contracts in a single atomic transaction. Think of it as a programmable router that can batch calls, manage approvals, and execute strategy logic on behalf of users, all while maintaining strict security boundaries.

The UPM serves as infrastructure for use cases such as:

* One-click leveraged positions (swap, stake, borrow in one tx)
* Zap contracts (enter complex strategies with one asset)
* Multi-protocol compositions (ORBT + external protocols)
* Gas optimization (batch operations)
* Flash loan integrations (arbitrage, liquidations)

Unlike smart contract wallets, UPM is stateless and permissionless, it does not hold funds between transactions and requires no setup. Frontends and power users compose workflows using low-level primitives (`doCall`, `doBatchCalls`) while respecting trust boundaries.
