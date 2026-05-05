---
hidden: true
---

# Pocket   Tech Dive

## 1) Summary

**Pocket** is the custody and execution layer that turns Orbit UCE into a high-throughput, yield-aware settlement engine. Each supported asset (e.g., USDC) has a **global pocket** and can additionally have **allocator-specific pockets**. Pockets custody users’ underlyings that arrive via swaps, invest them in vetted **money markets (Aave first)**, and stand ready—via allowances and/or **credit delegation**—to deliver instant liquidity for redemptions and intent-based settlements.

Key properties:

* **Direct custody:** Pockets are plain addresses (EOA, multisig, or strategy vault) that hold the underlying asset; UCE pulls funds via ERC‑20 allowances.
* **Low-latency settlements:** A per-asset **reserve buffer** sits on the UCE for everyday outflows; bursts are served by pocket pulls.
* **Yield routing:** Non-reserved balances are deposited into **Aave** to earn money-market yield while remaining withdrawable on demand.
* **Credit delegation:** Each pocket can delegate borrow capacity (Aave v3) to pre-approved settlement actors, enabling **instant intent settlements** without waiting for pocket withdrawals.
* **Attribution & incentives:** Referral mapping ties user flow to an allocator’s pocket and consumes that allocator’s reserved 0x inventory first—encouraging pre-provisioning.

***

## 2) What is a “pocket”?

In Orbit UCE, a **pocket** is “where the money waits”—the custody endpoint for each whitelisted stablecoin:

* **Address type:** EOA, multisig, or a minimal strategy vault controlled by governance or allocator policy.
* **Accounting role:**
  * **On UCE:** a per-asset **reserve buffer** (reserveBps) is retained on-hand.
  * **In the pocket:** the remainder of inbound user funds is deposited to Aave (aTokens accrue interest).
* **Permissions:** Pockets **approve** UCE to spend (pull) up to an allowance; UCE calls safeTransferFrom(pocket, UCE, amount) during redemptions.
* **Routing:** For user mints or redeems with a referral, UCE routes inflows to (and pulls from) the **referrer’s pocket** first; otherwise it uses the **global pocket**.

In short: **UCE handles quotes and burning/minting; pockets hold the underlyings and produce yield while staying permissioned for fast pulls.**

***

## 3) Necessity

* **UX:** Instant swaps/redemptions depend on reliable liquidity. Pockets deliver it with a two-tier design: **on-hand UCE buffer** for small/frequent flows, **pocket pulls** for bursts.
* **Economics:** Aave yield from pocket balances offsets system costs, sustains the treasury, and supports allocator carry.
* **Attribution & competition:** Allocators who pre-provision pockets win order flow (via referral attribution) and spreads—turning pockets into a competitive, monetizable surface.

***

## 4) Money markets in DeFi

A **money market** (Aave/Compound-style) is a pooled lending protocol:

* **Suppliers** deposit assets into a shared pool and receive **interest-bearing tokens** (e.g., Aave’s **aTokens**).
* **Borrowers** draw from the pool by posting collateral; interest paid by borrowers funds supplier yield.
* **Rates** float with **utilization**: as more of the pool is borrowed, borrow rates rise (often with a “kink” step) and supply rates follow.
* **Risk controls:** per-asset collateral factors (LTV), liquidation thresholds, reserve factors, supply/borrow caps, isolation lists, and (in Aave v3) features like **eMode** for correlated assets.
* **Composability:** aTokens are standard ERC‑20s accruing interest, usable in other protocols.

Important invariants:

* Liquidity is pooled: withdrawals succeed if pool liquidity is available; otherwise they may be rate-limited or partially filled until liquidity returns.
* Interest accrues continuously via indices maintained by the protocol; aTokens increase in value by balance growth (rebasing) or index scaling.

***

## 5) How Aave works

We operate in **Aave v3** (conceptually; chain-specific params vary):

* **Supply:** deposit USDC → receive **aUSDC**; balance grows as interest accrues.
* **Borrow:** a borrower takes USDC and receives **variable** or **stable** debt (stable often disabled for some stables).
* **Interest:** utilization-based curves with a kink; supply APR ≈ borrow APR × (1 − reserve factor) × utilization.
* **Risk settings:** per-asset **LTV**, **liquidation threshold**, **liquidation bonus**, **caps**; optional **eMode** (higher LTV for correlated assets).
* **Credit delegation:** the supplier authorizes a borrower to draw debt **against the supplier’s collateral** by calling approveDelegation() on Aave’s debt token. The borrower can then borrow() without posting its own collateral. If the borrower fails to repay, **the delegator’s collateral is at risk**.

Our initial scope is simple: pockets **supply only** (no native borrowing). We enable **credit delegation** to whitelisted settlement actors for instant pulls, with strict caps so HF never approaches liquidation.

***

## 6) Pocket lifecycle and flows

### 6.1 Inbound (stable → 0xAsset)

{% stepper %}
{% step %}
### User sends funds to UCE

User sends stablecoin **S** to UCE (swap in).
{% endstep %}

{% step %}
### Reserve retention and forwarding

UCE retains **reserveBps** (e.g., 25%) on-hand as **reserve**; forwards the remainder to the **selected pocket** (referrer’s or global).
{% endstep %}

{% step %}
### Pocket deposits and issuance

Pocket deposits the non-reserved balance to **Aave** (or keeps a tiny on-pocket buffer if configured). UCE swaps (or mints the shortfall) 0xAsset to the user (issuance path; no redemption-rate effects).
{% endstep %}
{% endstepper %}

***

### 6.2 Outbound (0xAsset → stable)

{% stepper %}
{% step %}
### User burns 0xAsset

User sends 0xAsset to UCE; UCE **burns** it.
{% endstep %}

{% step %}
### Payout from reserve, then pocket pull

UCE pays out **S** from on-hand reserve; if insufficient, it **pulls** from the pocket using its allowance (instant, subject to allowance and pocket balance).
{% endstep %}

{% step %}
### Pocket top-up

In the background, the pocket may top the UCE reserve back up.
{% endstep %}
{% endstepper %}

***

### 6.3 Intent-based settlement (instant)

{% stepper %}
{% step %}
### Solver needs asset immediately

A solver (whitelisted) receives an **intent** and needs **USDC now**.
{% endstep %}

{% step %}
### Pre-delegated credit on Aave

Pocket has pre-delegated borrow capacity on Aave’s **variable debt USDC** to the solver via approveDelegation.
{% endstep %}

{% step %}
### Borrow, settle, repay

Solver **borrows** from Aave instantly (credit backed by pocket’s aUSDC collateral), executes settlement, then the UCE/pocket **repays** the borrow from inflows or reserves shortly after.
{% endstep %}

{% step %}
### Post-trade risk handling

If not repaid promptly, the **debt persists**; limits/alerts kick in. Governance can revoke delegation immediately.
{% endstep %}
{% endstepper %}

This dual path—**allowance pulls** and **credit delegation**—makes pockets the **liquidity entry points** for high-speed settlement.

***

## 7) Aave-only pocket strategy

### 7.1 Objectives

* Maximize **availability** for redemptions and intents.
* Earn **conservative base yield** with minimal directional risk.
* Keep **risk of liquidation ≈ 0** by strict delegation caps.

### 7.2 Mechanics

* **Supply policy:**
  * Deposit all non-reserved USDC to Aave → receive **aUSDC**.
  * Optionally keep a small **on-pocket hot buffer** (e.g., 1–3% of pocket balance) to reduce borrow/withdraw round-trips for micro-settlements.
* **Withdrawal policy:**
  * For standard UCE redemptions, rely on **allowance pulls** (pocket → UCE).
  * If the pocket’s on-pocket buffer is used, **rebalance** by withdrawing from Aave back to pocket and/or refilling UCE reserve.
* **Credit delegation policy:**
  * Maintain **per-delegate allowances** on Aave variable-debt tokens: approveDelegation(delegate, limit).
  * Enforce **per-delegate ceilings**, **tenor limits** (time-boxed usage), and **global utilization caps** (e.g., “total delegated notional ≤ 25% of borrowable; target used ≤ 10%”).
  * Continuous **HF guardrails**: compute borrowable vs. delegated usage so **HF never < 2.0** under normal volatility.
* **Chain specifics:** Use per-chain Aave markets where liquidity is deep and risk params are conservative. Enable **eMode** for stable-stable if materially helpful and safe.

***

## 8) Credit delegation for instant settlements

### 8.1 What it is

**Aave credit delegation** lets a pocket (the **delegator**) that holds aTokens **authorize** another address (the **delegate**) to borrow without posting collateral. The debt is denominated in the borrowed asset (e.g., USDC) and is **secured by the delegator’s collateral**.

### 8.2 Why we use it

* **Latency:** The delegate can borrow and settle **in one transaction**—ideal for **intent-based settlement networks** (e.g., RFQ/solver systems).
* **Operational simplicity:** No pre-move needed from pocket to UCE; no allowance races; **no socialized slippage**—the delegate gets funds at par from Aave’s pool.
* **Determinism:** Delegation amounts, expiries, and per-delegate policies are on-chain and auditable.

### 8.3 How we keep it safe

* **Hard caps:** Per-delegate and global caps based on pocket’s collateral. Start small (e.g., **5–15%** of borrowable), increase with performance.
* **Tenor controls:** Allowances with **time locks** and **auto-expiry** (e.g., 30–120 minutes) for rapid settlement loops.
* **Kill switches:** Instant **revoke** (set delegation to 0), per-delegate pause, and global pause.
* **HF discipline:** Model worst-case utilization spikes and price shocks; set caps to keep **HF > 2.0** even under stress.
* **Escrowed repayment flows:** Where feasible, structure the solver’s flow so repayments are **atomic** with settlement proceeds or enforced by **post-trade bonds**.

### 8.4 Example (USDC)

{% stepper %}
{% step %}
### Pocket supplies collateral

Pocket supplies 10m USDC → holds 10m **aUSDC**.
{% endstep %}

{% step %}
### Governance sets caps

Governance sets: global delegation cap **1.5m**, per-solver cap **300k**, expiry **60 min**.
{% endstep %}

{% step %}
### Solver borrows and settles

Solver is authorized; borrows **$250k USDC** via Aave; completes an intent; returns **$250k + fee** within 10 minutes.
{% endstep %}

{% step %}
### Debt cleared, HF preserved

Pocket’s debt goes back to **0**; HF remains high throughout; interest earned on aUSDC continues uninterrupted.
{% endstep %}
{% endstepper %}

***

## 9) Interfaces & implementation patterns

The core Orbit UCE contract already supports pockets via pockets\[asset], per-allocator pocket mapping, and UCE-side **allowance pulls**. Below are thin adapters you’ll typically pair with a pocket.

### 9.1 Aave adapter (sketch)

Responsibilities:

* supply(asset, amount) → deposit to Aave, receive aTokens.
* withdraw(asset, amount) → redeem from Aave back to the pocket.
* **Query helpers:** aToken balance, liquidity index, available liquidity, HF projection.

### 9.2 Credit-delegation manager (sketch)

Responsibilities:

* Maintain **per-delegate limits** and **expiries**; call Aave approveDelegation on variable-debt tokens.
* Track **active delegated debt** per delegate; enforce **circuit breakers**.
* Expose revoke(delegate) and pause(); emit detailed events for monitoring.

### 9.3 Allowance & pull path

* Pockets set ERC‑20 approve(UCE, max) for the asset.
* UCE uses \_pushAssetFromContext to **pull** from pocket on redemptions (already implemented).

***

## 10) Reserve buffer & rebalancing

* **On-hand reserves (UCE):** Parameterized by reserveBps (typ. 15–35% for core stables).
* **Pocket buffer (optional):** Keep a small **hot float** (1–3%) to avoid withdraw/borrow on tiny bursts.
* **Rebalancing rules:**
  * If UCE reserve falls below a watermark, **pull** from pocket (or **withdraw** from Aave to pocket, then pull).
  * If UCE reserve is above a ceiling, **push** excess to pocket, then **supply** to Aave.

***

## 11) Comparison to allocator systems (context)

Like allocators in Sky’s SLL (Spark Liquidity Layer) with an ALM planner, pockets centralize **deployment** and **settlement readiness**: allocators manage custody and yield, while the engine preserves strict settlement determinism. The twist here is **credit delegation** as an express lane for intent-based networks.

***

## 12) Minimal policy defaults (recommended starting values)

* reserveBps (UCE on-hand): **25%** for USDC/USDT/DAI.
* Pocket hot buffer: **2%** of pocket balance.
* Delegation: global cap **≤ 15%** of borrowable; per-delegate **≤ 3%**; expiry **60 minutes**; HF floor **≥ 2.2** under worst-case stress.
* Alerts: UCE reserve < **10%** of 7-day p95 outflow; Aave utilization > **90%**; delegated usage > **50%** of cap; HF < **2.5** projected.

***

## 13) End-to-end example (numbers)

{% stepper %}
{% step %}
### Large mint flows to pocket

User swaps **$1,000,000 USDC → 0xUSD**.

* UCE keeps **$250k** on-hand (25%) and sends **$750k** to pocket.
* Pocket supplies **$750k** to Aave → receives **aUSDC**.
{% endstep %}

{% step %}
### Redemption uses reserve + pull

Later, a redemption for **$600k USDC** arrives.

* UCE pays **$250k** from reserve, then **pulls $350k** from pocket via allowance (near-instant).
{% endstep %}

{% step %}
### Concurrent intent via delegation

Meanwhile, a solver needs **$200k** for an intent.

* Pocket has a **$300k** delegation cap to this solver; solver borrows **$200k** from Aave **instantly**, settles, and repays within 15 minutes.
* The pocket remains fully collateralized; HF never approaches stress thresholds; aUSDC continues accruing yield.
{% endstep %}
{% endstepper %}

***

If you want, I can:

* Produce suggested Solidity interface sketches for the Aave adapter and credit-delegation manager.
* Convert any of the policy defaults into on-chain configurable parameters with types and ranges.
