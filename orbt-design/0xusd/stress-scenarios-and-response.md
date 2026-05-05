# Stress Scenarios and Response

#### 1) Depeg of an Underlying Stablecoin

* **Immediate**: Pause the asset or set H(asset) ≪ 1. Redemptions into that asset halt; redemptions out of it use haircut-aware quotes. Pockets unwind to safer stables per policy; UCE can `emergencyWithdraw` to Treasury if needed.
* **Peg effect**: Haircutting prevents a depegged asset from draining value from 0xUSD. Arbitrage routes to higher-quality stables, preserving the $1 target against the healthy part of the basket.

#### 2) Pocket Illiquidity / Allowance Lapses

* UCE spends on-hand reserves first; if insufficient and a pocket cannot be pulled, settlements may queue or revert. Monitoring on allowances and reserve depletion is required. Temporarily raising `reserveBps` increases on-hand buffers.

#### 3) 0xUSD Liquidity Run

* On issuance demand (stable → 0xUSD): UCE can mint to settle; no redemption-rate impact.
* On redemption demand (0xUSD → stable): The redemption rate rises (as a stress meter), and allocator debts scale to internalize system stress.

#### 4) Oracle Failure / Out-of-Band Prices

* Per-asset pause on staleness/deviation breach. Convertibility degrades to on-hand/pocket balances for unpaused assets; pricing defaults to conservative behavior rather than unsafe quotes.
