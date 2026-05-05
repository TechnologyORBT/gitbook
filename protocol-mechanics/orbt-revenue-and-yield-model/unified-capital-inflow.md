# Unified Capital Inflow

ORBT’s **Unified Capital Flow** framework forms the backbone of its liquidity architecture - enabling seamless aggregation, movement, and deployment of capital across chains, asset types, and liquidity layers.

### Cross-Asset Aggregation

ORBT aggregates liquidity from multiple asset classes into a **single composable liquidity layer**, unifying digital and institutional capital sources.\
This includes:

* **Stablecoins** (e.g., USDC, USDT, DAI, 0xUSD) for predictable settlement liquidity.
* **Blue-chip crypto assets** such as **BTC, ETH, SOL, and BNB** serving as high-value collateral and liquidity anchors.
* **Institutional liquidity** provided by permissioned entities - including treasury desks, payment processors, and corporate participants - who deposit via permissioned Pockets.

This aggregation creates a **universal capital base** that powers every function within the ORBT ecosystem, from stablecoin issuance to lending, settlement, and staking.

### Composable Vault Architecture

All assets within ORBT are held in **composable, chain-agnostic vaults**, governed by the [**Unified Collateral Engine**](../../orbt-design/uce/) **(UCE)** and integrated into the **Unified Collateral Engine(UCE)**.

Each vault follows the [ERC-4626](policy-governed-collateralization.md#segregated-collateral-pools) standard, ensuring:

* **Non-rehypothecated custody:** assets remain transparently owned by depositors.
* **Composability:** vaults can interact with Pockets, allocators, and other modules through standardized interfaces.
* **Chain Agnosticism:** vault logic remains uniform across EVM and non-EVM chains, with cross-chain liquidity routing managed via secure bridge frameworks (e.g., LayerZero, CCIP, CCTP).

This structure allows ORBT to **treat liquidity as a single programmable resource**, regardless of its origin chain or underlying asset.

### Capital Routing & Flow Control

Capital within ORBT flows through a **multi-layer routing system** that ensures optimal utilization, safety, and transparency:

1. **Deposit Phase:**\
   Users and institutions [deposit assets](../../user-flow-and-guides/how-to-buy-0xassets.md) into vaults corresponding to their preferred chain or collateral type.
2. **Aggregation Phase:**\
   The UCE aggregates all deposits into the [Unified Collateral Layer](../../orbt-design/uce/concepts/swaps.md#id-3-liquidity-sourcing-and-delivery), dynamically rebalancing positions across assets and chains based on liquidity needs and risk constraints.
3. **Deployment Phase:**\
   Surplus liquidity is deployed through the [**Modular Stablecoin Framework (MSF)**](../../security-and-safety/capital-protection.md#yield-optimization-via-msf) into low-risk, yield-generating strategies such as lending, staking, etc.
4. **Settlement Phase:**\
   When users execute payments, borrow, or redeem, the UCE ensures[ instant access](../../orbt-design/uce/concepts/reserve-policy.md) to required liquidity, powered by pre-funded reserves and bridge-agnostic transfer mechanisms.

This **closed-loop capital system** allows ORBT to maintain real-time solvency while supporting complex cross-chain operations without manual coordination.

### Chain-Agnostic Liquidity Model

ORBT’s chain-agnostic design abstracts away the complexities of fragmented liquidity.\
Through its bridging and oracle frameworks, capital flows seamlessly between ecosystems like Ethereum, BNB Chain, Solana, and Layer 2 networks, maintaining synchronized accounting via on-chain proofs.

* **Unified Minting:** 0xUSD and other assets can be minted natively on multiple chains under [allocator quotas](../../orbt-design/uce/concepts/allocators/referral-vs-non-referral.md).
* **Bridge Segmentation:** Each chain’s vaults are isolated; failures or congestion in one network do not compromise global liquidity.
* **Dynamic Balancing:** Automated rebalancing ensures that liquidity depth remains consistent across all supported ecosystems.

This model transforms ORBT into a **global liquidity fabric,** one that links DeFi ecosystems, institutional capital pools, and payment rails into a single continuous capital flow.

### Capital Efficiency and Safety

Every movement of capital within ORBT adheres to policy-defined thresholds for **reserve ratio, exposure limit, and collateral utilization**.

* The **UCE** enforces [1:1 hard peg](policy-governed-collateralization.md#id-1-1-collateralized-minting) and prevents [cross-pledging](../../orbt-design/uce/concepts/swaps.md#swap-planes-pair-gated-conversion) & maintains a liquidity buffer for instant settlement.
* The **MSF** deploys only approved surplus capital to maintain safety and performance balance.

This ensures that capital efficiency never comes at the expense of systemic safety or user protection.

### Transparency and Monitoring

ORBT provides **real-time capital flow visualization** through on-chain dashboards and subgraph integrations.\
Users, institutions, and governance participants can track:

* Total aggregated liquidity by asset type and chain.
* Collateral ratios and vault utilizations.
* Cross-chain transfer activity and reserve depth.

This radical transparency transforms capital flow into a measurable, auditable process, reinforcing trust at both institutional and retail levels.
