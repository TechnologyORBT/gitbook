# Overview

**Pocket** is the custody and execution layer that turns Orbt UCE into a high-throughput, yield-aware settlement engine. Each supported asset (e.g., USDC) has a **global pocket** and can additionally have **allocator-specific pockets**. Pockets custody users’ underlyings that arrive via swaps, invest them in vetted [**money markets**](what-is-a-money-market/) **(Aave first)**, and stand ready, via allowances and/or [**credit delegation**](credit-delegation-for-instant-settlements/)**,** to deliver instant liquidity for redemptions and intent-based settlements.

Key properties:

* **Direct custody:** Pockets are plain addresses (EOA, multisig, or strategy vault) that hold the underlying asset; UCE pulls funds via ERC‑20 allowances.
* **Low-latency settlements:** A per-asset [**reserve buffer**](../uce/concepts/reserve-policy.md) sits on the UCE for everyday outflows; bursts are served by pocket pulls.
* **Yield routing:** Non-reserved balances are deposited into [**Aave** ](aave-only-pocket-strategy/)to earn money-market yield while remaining withdrawable on demand.
* **Credit delegation:** Each pocket can [delegate borrow capacity](credit-delegation-for-instant-settlements/) (Aave v3) to pre-approved settlement actors, enabling **instant intent settlements** without waiting for pocket withdrawals.
* **Attribution & incentives:** Referral mapping ties user flow to an allocator’s pocket and consumes that allocator’s reserved 0x inventory first, encouraging pre-provisioning.
