# 0xUSD

## Overview

0xUSD is a synthetic, USD-targeted unit issued by Orbt’s Unified Collateral Engine ([UCE](../uce/)). The peg is maintained by&#x20;

1. oracle-aware convertibility between 0xUSD and a diversified basket of USD-stable underlyings,
2. unified settlement against pooled inventories and pockets,
3. credit-based issuance by permissioned Allocators with explicit limits,
4. a dynamic redemption rate that is applied when users convert from 0xUSD to underlying, and
5. proportional debt scaling that socializes tail losses across indebted Allocators.&#x20;

This note specifies the peg mechanics, incentives, and stress response when supporting “all stablecoins” as underlying capital:&#x20;

* **Target**: 0xUSD ≈ $1.00 (hard 1:1 pegged; conversions are oracle-driven and haircut-aware).
* **Convertibility**: Users can swap any supported stablecoin ↔ 0xUSD, or 0xUSD ↔ any supported stablecoin, through UCE at oracle quotes with conservative rounding.
* **Inventory Discipline:** On-hand reserves + pockets (global and per-allocator) supply settlement. Allocators hold 0xUSD inventory (`reservedZeroX`) and maintain underlying liquidity in pockets that can be deployed into strategies.
* **Stress Hygiene**: On 0xUSD → underlying redemptions, a time-varying redemption rate updates and allocator debts may be scaled down to socialize loss.\
  Asset Breadth: Governance whitelists stablecoins (USDC, USDT, DAI, LUSD, FRAX, sDAI, etc.). Each asset has `reserveBps`, a pause switch, and (future) per-asset haircut/weight via oracle policy.

***

### System Actors and Inventory Context

* **Users**: Swap between 0xUSD and supported stablecoins; optionally interact with s0xUSD (ERC-4626 wrapper on 0xUSD).
* **Allocators**: Permissioned market-makers with a line of credit (ceiling, dailyCap) to mint 0xUSD, warehouse it as reservedZeroX, and manage per-asset pockets that deploy supported stablecoins into external strategies (Aave, Maker DSR/sDAI, T-bill wrappers, Curve pools, etc.).
* **UCE (the engine)**: The settlement hub. Holds on-hand 0xUSD and on-hand underlyings (reserve buffer) and routes to pockets via allowances. Updates the dynamic redemption rate and performs debt scaling on 0xUSD → underlying conversions.
* **Treasury**: Receives allocator borrow fees (assessed on allocator repay).

***

### Pricing and Quotes

Let P(asset) be the USD price from the oracle (with staleness/deviation guards).&#x20;

Let H(asset) ∈ \[0,1] be a governance haircut (0 = no credit; 1 = par). Quotes follow:

$$
amountOut = amountIn * (P(in) * H(in)) / (P(out) * H(out)) * 10^Δdecimals
$$

{% hint style="info" %}
Conservative rounding applies (round down on payouts in 0xUSD terms; round up on liabilities).
{% endhint %}

***

### Peg Mechanics: Primary and Secondary Paths

#### A) 0xUSD Premium (> $1 externally)

Arb supply expands.

1. Allocators mint 0xUSD via credit lines (no redemption-rate effect at issuance) to build inventory and sell externally above $1, then later repay with supported stablecoins (paying borrowFeeBps). \
   `Profit = market premium − financing costs`.
2. Users swap stablecoins → 0xUSD at oracle quotes. If on-hand + reserved inventory is insufficient, UCE mints 0xUSD to the receiver to settle the trade. No redemption rate is applied on this path (issuance path).\
   → Both flows increase circulating 0xUSD where it trades rich, pushing price back toward $1.

#### B) 0xUSD Discount (< $1 externally)

Arb demand retires supply.

1. Users buy 0xUSD cheap externally and swap 0xUSD → stablecoins at UCE near $1 (oracle quote). UCE burns inbound 0xUSD and pays out supported stablecoins from on-hand reserves and pockets. This path updates the dynamic redemption rate and can trigger allocator debt scaling (see below).
2. Allocators buy back 0xUSD at a discount, reduce `reservedZeroX`, and/or repay their debt with stablecoins.\
   → Circulating 0xUSD shrinks; price lifts toward $1.

**Result**: Convertibility bands (plus rounding) create two-sided arbitrage that pins 0xUSD near $1 in normal liquidity.

***

### Routing, Pockets, and Reserves

* **Pockets**: An asset’s pocket is an address (EOA, multisig, or strategy vault) that custodies that stablecoin. There is a global pocket per asset and optional allocator-specific pockets. Pockets can deploy to low-volatility yield. UCE relies on ERC-20 allowances to pull funds for settlement.
* **Reserve Buffer** (reserveBps per asset): A fraction of inbound stablecoin is retained on UCE as on-hand buffer to satisfy small/fast outflows without pocket pulls (latency minimization). The remainder is forwarded to the selected pocket (allocator pocket if the caller is an allocator or referral maps; otherwise the global pocket).
* **Referral Attribution**: A user-supplied referral code can map to an allocator; inbound stablecoins then route to that allocator’s pocket and, when paying out 0xUSD, UCE first consumes that allocator’s `reservedZeroX`. This lets allocators monetize flow and encourages pre-provisioning of inventory.

#### Redemption Rate (0xUSD → Underlying) and Shortfall Socialization

Fix applied: The redemption rate is applicable when users swap from 0xUSD to underlying. It is not applied when users go from underlying to 0xUSD.

#### Dynamic Redemption Rate (update on redemption)

* Triggered only on 0xUSD → underlying conversions.
* The engine updates baseRedemptionRate proportionally to the redeemed fraction of total 0xUSD supply and decays it over time (\~0.5% hourly, capped, e.g., 5%).

Economically, it is a stress meter: heavy redemptions raise the rate; quiet periods let it decay.

#### Debt Scaling (proportional socialization onto issuers)

* On redemptions, UCE burns the inbound 0xUSD and pays out the underlying at the oracle quote.
* UCE then adjusts the global debt index so that indebted Allocators absorb systemwide tail losses pro-rata to their effective debt:\
  \
  $$newIndex = oldIndex * (EffTotal - NetShortfall) / EffTotal$$\
  \
  &#xNAN;_&#x77;here EffTotal is the sum of effective allocator debts_ and\
  &#xNAN;_&#x4E;etShortfall is a redemption-rate-adjusted amount derived from the 0xUSD redeemed._\
  &#xNAN;_&#x49;f NetShortfall ≥ EffTotal, the epoch is wiped and debts are reset (with index re-initialized)._
* `User experience`: the redeemer receives the quoted underlying; the redemption rate does not haircut the user’s payout. The cost of stress is internalized by issuers via the index scaling.

#### No Interest Rate on Issuance (underlying → 0xUSD)

* When users bring in stablecoins for 0xUSD and on-hand + reserved inventories are insufficient, UCE mints the shortfall 0xUSD to the receiver directly. No redemption rate or debt scaling is applied on this issuance path.

### Allocator Incentives and Risk

* **Carry Model**: Allocators earn external spread when selling 0xUSD rich and can farm stablecoin yields in pockets. They pay borrow fees (borrowFeeBps) when repaying. Credit ceilings and daily caps bound exposure and issuance velocity.
* **Inventory Obligation:** Serving referred flow requires maintaining `reservedZeroX` and pocket liquidity; failing to pre-provision pushes users into engine minting on the issuance path (still seamless for users) but leaves allocators exposed to future redemption-driven debt scaling.
* **Strategy Choice:** Pockets should favor low-volatility, short-exit-time venues; allowances and buffers must be sized so UCE pulls settle promptly.

### Supporting “All Stablecoins”

Governance whitelists assets with per-asset controls:

* **Onboarding tuple**: (asset, family, pocket, reserveBps). For 0xUSD we admit multiple USD stables and wrappers.
* **Haircuts and Pauses**: Quotes incorporate both P(asset) and H(asset). During a depeg, governance can reduce H(asset) (or pause the asset), preventing value leakage from 0xUSD holders into a depegged underlying.
* **Concentration and Caps**: Per-asset effective capacity prevents over-exposure. When a cap is hit, governance can slow routing by increasing reserveBps elsewhere or pausing new deposits.

### Steady-State Peg Dynamics

* **Narrow band:** In liquid conditions, arbitrage via UCE quotes keeps the external price of 0xUSD near $1 net of rounding. The reserve buffer handles routine outflows; pockets handle bursts; Allocators expand/contract supply as spreads dictate.
* **Premium episodes:** Allocators mint and sell; users can mint via underlying → 0xUSD with no redemption-rate impact; supply rises and premium compresses.
* **Discount episodes:** Users redeem 0xUSD → stablecoins; UCE burns 0xUSD, updates the redemption rate, and may scale allocator debts; supply falls and price lifts.

## Stress Scenarios and Response

#### 1) Depeg of an Underlying Stablecoin

* Immediate: Pause the asset or set H(asset) ≪ 1. Redemptions into that asset halt; redemptions out of it use haircut-aware quotes. Pockets unwind to safer stables per policy; UCE can emergencyWithdraw to Treasury if needed.
* Peg effect: Haircutting prevents a depegged asset from draining value from 0xUSD. Arbitrage routes to higher-quality stables, preserving the $1 target against the healthy part of the basket.

#### 2) Pocket Illiquidity / Allowance Lapses

* UCE spends on-hand reserves first; if insufficient and a pocket cannot be pulled, settlements may queue or revert. Monitoring on allowances and reserve depletion is required. Temporarily raising reserveBps increases on-hand buffers.

#### 3) 0xUSD Liquidity Run

* On issuance demand (stable → 0xUSD): UCE can mint to settle; no redemption-rate impact.
* On redemption demand (0xUSD → stable): The redemption rate rises (as a stress meter), and allocator debts scale to internalize system stress.

#### 4) Oracle Failure / Out-of-Band Prices

* Per-asset pause on staleness/deviation breach. Convertibility degrades to on-hand/pocket balances for unpaused assets; pricing defaults to conservative behavior rather than unsafe quotes.

### Flow of Funds&#x20;

* Stablecoin → 0xUSD (issuance path):\
  User sends stable S → UCE retains `reserveBps%` on-hand; forwards the remainder to the selected pocket (allocator’s pocket if caller/referral maps; else global).\
  Outbound 0xUSD: spend on-hand → allocator `reservedZeroX` (if allocator/referral) → unreserved on-hand → if still short, mint 0xUSD shortfall to the receiver.\
  No redemption rate or debt scaling applies here.
* 0xUSD → Stablecoin (redemption path):\
  User sends 0xUSD → UCE burns it.\
  Outbound S: spend on-hand reserves (decrement per-asset on-hand trackers) → pull remainder from appropriate pocket (allowance-bound) → transfer to user at oracle quote.\
  Then: UCE updates the redemption rate and may scale allocator debts (pro-rata via the global debt index).\
  User payout is not haircut by the redemption rate.
* s0xUSD wrapper: Deposit 0xUSD to receive shares; redeem shares for 0xUSD. The wrapper does not change peg logic or redemption-rate semantics; it only routes 0xUSD yield.

***

### Parameterization and Monitoring

* `reserveBps`: 15–35% typical for core stables; higher during volatile regimes.
* `MAX_REDEMPTION_RATE`: 3–5% ceiling; `DECAY_CONSTANT` tuned to \~0.5%/hour effective decay.
* Allocator lines: ceiling sized to allocator capitalization and track record; dailyCap limits issuance velocity.
* Haircuts H(asset): governance-set; updated for depeg risk, blacklistability, liquidity, and backing.
* Alerts: reserve depletion, pocket allowance low, surge in redemption rate, fast moves in debtIndex, oracle staleness/deviation.

### Why This Holds the Peg

* Hard Convertibility: Oracle-priced, haircut-aware swaps create an always-on primary market. Discounts are corrected by redemptions (supply burns with internalized cost to issuers). Premiums are corrected by allocator issuance and user mint-swaps (no redemption-rate friction on issuance).
* Distributed Liquidity: Unified inventory (on-hand + allocator `reservedZeroX` + pockets) reduces fragmentation. Referral attribution binds user flow to allocators who pre-provision, making the primary market competitive.
* Explicit Tail-Loss Sink: Debt scaling places systemic shortfall where it belongs - on the issuers (allocators) in proportion to their open credit - avoiding ambiguous bail-ins and strengthening creditor discipline.
