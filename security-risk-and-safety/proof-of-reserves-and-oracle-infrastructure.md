# Proof-of-Reserves & Oracle Infrastructure

#### **Audits and Formal Verification**

All core smart contracts of ORBT (**UCE, UPM, USM, Pocket templates, ORBTMM, token contracts**, etc.) undergo extensive security audits by reputable third-party firms.\
The ORBT team committed to **multiple independent audits** prior to mainnet launch.

For example, the **UCE and financial modules** might be audited by a firm experienced in DeFi lending protocols, while **cross-chain components** might be audited by a specialist in bridge security.\
Audit reports are made public for transparency.

Additionally, **critical components** are considered for **formal verification** – meaning the use of mathematical methods to prove correctness of certain properties (e.g., that collateral accounting always remains ≥ liabilities, or that the UCE cannot be drained through an arithmetic error).

Where formal verification is feasible (perhaps for the simpler stablecoin and UCE logic), it will be pursued to increase confidence, similar to how **MakerDAO** formally verified parts of their collateral engine.

#### **Battle-Tested Modules**

ORBT doesn’t reinvent the wheel unnecessarily – it leverages established designs where possible.

For instance:

* The **Unified Collateral Engine** concept is drawn from **Maker’s PSM**, which has been live and hardened.
* The **facilitator model** is similar to **Aave’s GHO design**, which was audited and launched.

By building on known patterns and standards (**ERC-20** for tokens, proven libraries for math like **OpenZeppelin’s**, etc.), ORBT reduces the surface for novel bugs.

That said, any modifications (like multi-collateral support in UCE or cross-chain aspects) are carefully tested.

#### **Oracle Reliability**

To prevent oracle manipulation and flash loan exploits, ORBT integrates:

* **Chainlink price feeds** for trusted off-chain data,
* **Oracle Security Modules (OSMs)** that delay updates (e.g., 1-hour lag) to mitigate short-term attacks,
* **On-chain sanity checks** before collateral valuations or liquidation events.

Pockets using market data (e.g., DEX prices for arbitrage) are isolated — their risks affect only their own outcomes, not protocol solvency.

#### **Re-entrancy and Contract Safety**

ORBT contracts adhere to known **best practices against re-entrancy**, using **mutex locks** or the **checks-effects-interactions pattern**.

Where applicable, **pull-based external calls** are used to avoid allowing external contract behavior to cascade back unexpectedly.

For example, if a Pocket interacts with an external protocol, the integration is carefully audited to ensure that external call cannot somehow re-enter ORBT’s core with inconsistent state.

On the oracle side, any **price, rate, or external data** feeding the **UCE** is handled via an **Oracle Security Module (OSM)** pattern (similar to Maker’s, which delays price updates by an hour to mitigate manipulation).

ORBT could similarly use a combination of **Chainlink feeds** (for reliable prices) and an **OSM delay** for critical prices (like ETH/USD for collateral valuation), so that even if an oracle is manipulated briefly, there’s time to react.

Pockets that rely on DEX prices (like an arbitrage Pocket) don’t feed into the system’s accounting except for profit calculation — risk there is more on the user side (they might not profit as expected if prices moved).
