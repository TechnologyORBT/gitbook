# Onboarding a new Allocator (end-to-end)

Goal: grant an address the Allocator role, set its credit line & fee, assign pockets per asset, and produce a referral code.

TL;DR: Register action type → register UCE as governed (one-time) → queue SET\_ALLOCATOR with parameters → wait timelock → execute → (optionally) update pockets.

{% stepper %}
{% step %}
### Prerequisites (usually one-time)

* Register UCE as a governed contract
  * Action: REGISTER\_CONTRACT
  * Payload: (address contractAddr, string name) → e.g., (UCE, "Orbit UCE")
  * Sign (≥ 4), queue, timelock, execute.
* Register product action types (if not already)
  * Actions: REGISTER\_ACTION\_TYPE for:
    * ACT\_SET\_ALLOCATOR
    * ACT\_SET\_ALLOCATOR\_POCKETS
  * Each call: payload (bytes32 actionType, string description).
{% endstep %}

{% step %}
### Parameters required for onboarding

* Allocator address (init.allocator)
* Credit line:
  * ceiling — max effective debt
  * dailyCap — max issuance per 24h rolling UTC day
* Borrow fee: borrowFeeBps (e.g., tuned to target \~5% annualized policy)
* Allowed flag: allowed = true
* Per-asset scope:
  * assets\[] — list of supported underlyings this allocator will manage
  * pocketsNew\[] — pocket addresses per asset (same order as assets\[])

The UCE also computes and assigns a referral code to the allocator during SET\_ALLOCATOR. The code is derived from (allocator, ceiling, dailyCap, borrowFeeBps). If these values change later, governance may re-assign a new code (the old mapping is cleared).
{% endstep %}

{% step %}
### The governance call(s)

1. Onboard / set allocator
   * Action: ACT\_SET\_ALLOCATOR
   * Payload: (op, SetAllocatorMemory init, address\[] assets, address\[] pocketsNew) where:
     * op = SET\_ALLOCATOR
     * init = { allocator, allowed, line{ceiling, dailyCap, mintedToday=0, lastMintDay=current}, borrowFeeBps }
     * assets\[] and pocketsNew\[] can be empty if pockets will be set in a follow-up call.
   * Effects (on UCE):
     * Marks allowed = true.
     * Stores credit line and borrowFeeBps.
     * Generates and maps a referralCode.
     * Emits AllocatorSet, CreditLineUpdated, AllocatorBorrowFeeSet, AllocatorReferralSet.
2. (Optional) Set or update allocator pockets
   * Action: ACT\_SET\_ALLOCATOR\_POCKETS
   * Payload: (allocator, asset, newPocket) for single-asset tweaks, or call ACT\_SET\_ALLOCATOR with op = UPDATE\_POCKET and arrays for batch updates.
   * Effects:
     * Updates allocator → asset → pocket mapping.
     * Emits AllocatorPocketSet and migrates any pullable balances between old/new pockets.

Both actions go through: sign (≥ 4, ≤ 9) → queue (timelock) → execute. If the signers or parameters are invalid, the governance module rejects them upfront (before queueing) by verifying valid action type, signatures, and nonce.
{% endstep %}

{% step %}
### After onboarding: operational readiness checklist

* Fund pockets with initial liquidity; keep allowances to UCE high enough for expected redemptions.
* Pre-mint inventory using allocatorCreditMint (staying within dailyCap and ceiling).
* Distribute referral code to direct your user flow.
* Set up monitoring for: reserve levels, allowances, pocket balances, Aave utilization, surcharge spikes, and debtIndex moves.
{% endstep %}
{% endstepper %}
