# Onboarding a new Allocator (end-to-end)

**Goal**: grant an address the Allocator role, set its credit line & fee, assign pockets per asset, and produce a referral code.

**TL;DR**

> Register action type → register UCE as governed (one-time) → queue `SET_ALLOCATOR` with parameters → wait timelock → execute → (optionally) update pockets.

{% stepper %}
{% step %}
### Prerequisites (usually one-time)

* Register UCE as a governed contract
  * Action: `REGISTER_CONTRACT`
  * Payload: (address contractAddr, string name) → e.g., (UCE, "Orbit UCE")
  * Sign (≥ 4), queue, timelock, execute.
* Register product action types (if not already)
  * Actions: `REGISTER_ACTION_TYPE` for:
    * `ACT_SET_ALLOCATOR`
    * `ACT_SET_ALLOCATOR_POCKETS`
  * Each call: `payload (bytes32 actionType, string description)`.
{% endstep %}

{% step %}
### Parameters required for onboarding

* Allocator address (init.allocator)
* Credit line:
  * ceiling - max effective debt
  * dailyCap - max issuance per 24h rolling UTC day
* Borrow fee: `borrowFeeBps` (e.g., tuned to target \~5% annualized policy)
* Allowed flag: allowed = true
* Per-asset scope:
  * `assets[]` - list of supported underlyings this allocator will manage
  * `pocketsNew[]` - pocket addresses per asset (same order as assets\[])

The UCE also computes and assigns a referral code to the allocator during `SET_ALLOCATOR`. The code is derived from (allocator, ceiling, dailyCap, borrowFeeBps). If these values change later, governance may re-assign a new code (the old mapping is cleared).
{% endstep %}

{% step %}
### The governance call(s)

1. Onboard / set allocator
   * Action: `ACT_SET_ALLOCATOR`
   * Payload: (`op`, `SetAllocatorMemory init`, `address[] assets`, `address[] pocketsNew`) where:
     * op = `SET_ALLOCATOR`
     * init = { `allocator`, `allowed`, `line`{`ceiling`, `dailyCap`, `mintedToday=0`, `lastMintDay=current`}, `borrowFeeBps` }
     * `assets[]` and `pocketsNew[]` can be empty if pockets will be set in a follow-up call.
   * Effects (on UCE):
     * Marks allowed = true.
     * Stores credit line and `borrowFeeBps`.
     * Generates and maps a [`referralCode`](../uce/concepts/allocators/referral-vs-non-referral.md).
     * Emits `AllocatorSet`, `CreditLineUpdated`, `AllocatorBorrowFeeSet`, `AllocatorReferralSet`.
2. (Optional) Set or update allocator pockets
   * Action: `ACT_SET_ALLOCATOR_POCKETS`
   * Payload: (`allocator, asset, newPocket`) for single-asset tweaks, or call `ACT_SET_ALLOCATOR` with `op = UPDATE_POCKET` and arrays for batch updates.
   * Effects:
     * Updates allocator → asset → pocket mapping.
     * Emits `AllocatorPocketSet` and migrates any pullable balances between old/new pockets.

Both actions go through: sign (≥ 4, ≤ 9) → queue (timelock) → execute. If the signers or parameters are invalid, the governance module rejects them upfront (before queueing) by verifying valid action type, signatures, and nonce.
{% endstep %}

{% step %}
### After onboarding: operational readiness checklist

* Fund pockets with initial liquidity; keep allowances to UCE high enough for expected redemptions.
* Pre-mint inventory using `allocatorCreditMint` (staying within dailyCap and ceiling).
* Distribute referral code to direct your user flow.
* Set up monitoring for: reserve levels, allowances, pocket balances, Aave utilization, surcharge spikes, and debtIndex moves
{% endstep %}
{% endstepper %}
