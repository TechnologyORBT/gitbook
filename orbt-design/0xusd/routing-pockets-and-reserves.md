# Routing, Pockets, and Reserves

* **Pockets**: An asset’s pocket is an address (EOA, multisig, or strategy vault) that custodies that stablecoin. There is a global pocket per asset and optional allocator-specific pockets. Pockets can deploy to low-volatility yield. UCE relies on ERC-20 allowances to pull funds for settlement.
* **Reserve Buffer** (reserveBps per asset): A fraction of inbound stablecoin is retained on UCE as on-hand buffer to satisfy small/fast outflows without pocket pulls (latency minimization). The remainder is forwarded to the selected pocket (allocator pocket if the caller is an allocator or referral maps; otherwise the global pocket).
* **Referral Attribution**: A user-supplied referral code can map to an allocator; inbound stablecoins then route to that allocator’s pocket and, when paying out 0xUSD, UCE first consumes that allocator’s reservedOx. This lets allocators monetize flow and encourages pre-provisioning of inventory.
