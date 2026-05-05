---
description: User Position Manager
---

# UPM

The **User Position Manager (UPM)** functions as the **controller** for all user-level interactions within ORBT’s core system. Whenever a user, integrator or allocator interacts with the protocol - such as [depositing collateral](../../user-flow-and-guides/how-to-buy-0xassets.md), minting [**0xAssets**](../0xassets.md), or initiating a [**Pocket** ](../pocket/)strategy.

{% hint style="success" %}
The UPM provides a single abstraction layer for all user actions across modules. This eliminates the need for users to interact directly with the underlying contracts managing collateral, debt, or yield.
{% endhint %}

The UPM abstracts complexity for end users by serving as a unified control layer. _For instance_, when a user deposits **USDT** and mints **0xUSD**, the UPM creates a position that links the USDT (tracked by the [**Unified Collateral Engine**](../uce/)) with the corresponding 0xUSD debt. Through this mechanism, users can manage or close positions seamlessly through a single interface, without needing to directly interact with multiple contracts or modules.

Functionally, the UPM is analogous to an **account management module** or **MakerDAO’s CDP/Vault Manager**, but with expanded capabilities. It not only maintains user balances and liabilities but also supports **dynamic actions**, such as deploying liquidity into Pockets or adjusting yield strategies in real time.

By consolidating position tracking and activity management, the UPM provides users with a **comprehensive dashboard** that displays:

* Collateral deposited
* Assets borrowed
* Liquidity deployed into Pockets
* Yield accrued or claimed

Even when actions span multiple chains or modules, the UPM ensures a unified, auditable view of user activity - significantly improving usability, transparency, and oversight of complex multi-chain strategies.

***

The **User Position Manager** acts as the control layer that ties together user activity, collateral management, and strategy execution. By providing a unified, auditable record of all positions, it ensures transparency, composability, and ease of interaction across ORBT’s modular ecosystem.
