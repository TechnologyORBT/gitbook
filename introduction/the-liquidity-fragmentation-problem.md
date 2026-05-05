---
description: '|| the market pays fees; users get none ||'
---

# The Liquidity Fragmentation Problem

**What’s broken today**

* **Liquidity is scattered** across chains, AMMs, bridges, CEXs, and custodians → shallow pools, slow routing, and forced **pre-funding**.
* **Every hop takes a cut:** users pay **slippage + venue fees** that vanish into spreads, MEV, and tolls; nothing flows back to holders.
* **No predictable yield:** LPs face impermanent loss and venue risk; treasuries leave cash idle to “be safe.”

To move value, users and institutions **pre-fund** routes, then pay slippage and fees each hop. It’s fast(ish), but capital-inefficient and the fee stream simply **evaporates** into LP spreads, MEV, and venue tolls.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

**Net effect:** hundreds of billions pay for liquidity access, yet **no unified layer** captures it for users. Capital sits pre-positioned and idle between moves; fees are paid, value is lost.

#### Why fragmentation hurts

* **Pre-funding everywhere:** Treasuries keep floats on multiple chains/venues “just in case.” Idle cash earns nothing.
* **Every hop charges you:** Slippage, spreads, routing fees, bridge tolls, and MEV.
* **Slow refunds of liquidity:** After a transfer, capital trickles back—not instantly—so you over-fund the next transfer too.
* **No user dividend:** The fee stream funds everyone _except_ the asset holder.

## Centralized Stablecoins & Custodial Rails

USDC/USDT provide deep liquidity and tight pegs, but users inherit custodial concentration and policy risk while issuers internalize yield on backing. Liquidity is siloed by chain and venue, forcing treasuries to pre-fund multiple rails and keep idle floats. Fees from swaps, bridges, and settlements accrue to intermediaries and market venues—not to the end holders funding the activity—so dollars parked in centralized rails are typically “return-free” for users despite heavy system usage.

## Decentralized Stablecoins, AMMs & LP Yield

On-chain designs offer transparency and composability, yet scale and durability are inconsistent. Over-collateralized models grow with leverage demand and stall in risk-off regimes; algorithmic designs have shown fragility and depeg cascades. AMM liquidity exposes LPs to impermanent loss and venue risk, producing volatile, path-dependent returns. Emission-based incentives decay over time, and most fee capture remains pool- or router-level, leaving typical users without a predictable, defensible savings outcome.

***

## The Settlement-Powered Money Layer

|| plug into the fees, not around them ||

Pre-funded settlements let ORBT unify fragmented rails behind 0xAssets, creating a single, chain-agnostic experience layer.\
By monetizing in-transit capital and cycling it back within blocks, ORBT delivers real, on-chain yield to S0xAsset holders while defending a 1:1 peg.
