# Allocator roles & responsibilities

{% stepper %}
{% step %}
### Liquidity provisioning

* Maintain `reservedZeroX` to serve your referred users without delay.
* Pre-fund pockets and monitor [UCE](../uce/) `reserveBps` thresholds to avoid slow pulls.
{% endstep %}

{% step %}
### Custody & security

* Use hardened multisigs/EOAs for pockets; rotate keys; enforce internal controls.
* Keep ERC-20 allowances from pockets to UCE sufficient for expected bursts (and monitor them).
{% endstep %}

{% step %}
### Yield management (conservative)

* Deploy pocket balances into approved [money markets](../pocket/what-is-a-money-market/) ([Aave](../pocket/what-is-a-money-market/how-aave-works.md) first).
* Keep a small on-pocket [hot buffer](../pocket/reserve-buffer-and-rebalancing.md) (e.g., 1–3%) for micro-settlements; avoid over-deploying.
{% endstep %}

{% step %}
### Credit usage discipline

* Respect `ceiling` and `dailyCap`; monitor `mintedToday` rollovers by UTC day.
* Plan issuance vs. redemptions to minimize financing cost and avoid scarcity fees.
{% endstep %}

{% step %}
### Repayment operations

* Regularly repay with underlyings to manage debt and fees.
* Keep treasury fee implications (`borrowFeeBps`) in mind when choosing which asset to repay with.
{% endstep %}

{% step %}
### Flow attribution readiness

* Promote and distribute your [referral code](../uce/concepts/allocators/referral-attribution-and-inventory-consumption.md); ensure your pockets and `reservedZeroX` are ready to serve that flow.
{% endstep %}

{% step %}
### Risk & incident response

* Act on asset pauses or haircut policy changes (e.g., depegs).
* Be prepared to unwind/withdraw from strategies quickly when governance signals risk.
{% endstep %}

{% step %}
### Monitoring & reporting

Track: your `allocatorDebt()`, `reservedZeroX`, pocket balances & allowances, Aave utilization, UCE reserve levels, redemption surcharge, and debtIndex moves.

Set alerts for: low reserve/allowance, high market utilization, governance timelock events, and any unfilled pulls.
{% endstep %}
{% endstepper %}
