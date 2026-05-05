# Intent-based settlement (instant)

{% stepper %}
{% step %}
### Solver needs asset immediately

A solver (whitelisted) receives an **intent** and needs **USDC now**.
{% endstep %}

{% step %}
### Pre-delegated credit on Aave

Pocket has pre-delegated borrow capacity on Aave’s **variable debt USDC** to the solver via `approveDelegation`.
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

This dual path, **allowance pulls** and **credit delegation,** makes pockets the **liquidity entry points** for high-speed settlement.
