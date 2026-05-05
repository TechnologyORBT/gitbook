# Overview

s0xBTC, s0xETH, s0xUSD). Built on the ERC-4626 tokenized vault standard, the USM enables users to deposit 0x assets and receive interest-bearing shares that appreciate in value over time through a governance-controlled interest rate mechanism.

The USM implements an accumulator-based savings model inspired by MakerDAO's DSR/sUSDS, where a global exchange rate grows continuously based on a per-second interest rate. Key advantages:

* Gas Efficiency: interest accrual is computed algorithmically without per-user updates
* Composability: full ERC-4626 compatibility integrates with DeFi protocols
* Dual Yield Mechanism: native interest accrual + optional external reward token emissions
* Governance Control: interest rates adjustable via multisig/timelock governance
* Flexible Exit: configurable withdrawal limits and unstake delays
* Rewards-Only Mode: opt to accrue only external rewards (no minting of underlying)

s0xAsset tokens are ERC-20 claims on a growing pool of underlying 0x assets. Users can enter/exit (subject to delays) and shares appreciate via the global interest rate.
