# Parameterization and Monitoring

* reserveBps: 15–35% typical for core stables; higher during volatile regimes.
* MAX\_REDEMPTION\_RATE: 3–5% ceiling; DECAY\_CONSTANT tuned to \~0.5%/hour effective decay.
* Allocator lines: ceiling sized to allocator capitalization and track record; dailyCap limits issuance velocity.
* Haircuts H(asset): governance-set; updated for depeg risk, blacklistability, liquidity, and backing.
* Alerts: reserve depletion, pocket allowance low, surge in redemption rate, fast moves in debtIndex, oracle staleness/deviation.

### Why This Holds the Peg

* Hard Convertibility: Oracle-priced, haircut-aware swaps create an always-on primary market. Discounts are corrected by redemptions (supply burns with internalized cost to issuers). Premiums are corrected by allocator issuance and user mint-swaps (no redemption-rate friction on issuance).
* Distributed Liquidity: Unified inventory (on-hand + allocator reservedOx + pockets) reduces fragmentation. Referral attribution binds user flow to allocators who pre-provision, making the primary market competitive.
* Explicit Tail-Loss Sink: Debt scaling places systemic shortfall where it belongs—on the issuers (allocators) in proportion to their open credit—avoiding ambiguous bail-ins and strengthening creditor discipline.
