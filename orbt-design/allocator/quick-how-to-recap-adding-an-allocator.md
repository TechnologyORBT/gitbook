# Quick “how-to” recap: adding an Allocator

{% stepper %}
{% step %}
(One-time) Register UCE as governed (`REGISTER_CONTRACT`).
{% endstep %}

{% step %}
(One-time) Register action types for UCE (`REGISTER_ACTION_TYPE` for `ACT_SET_ALLOCATOR` and `ACT_SET_ALLOCATOR_POCKETS`).
{% endstep %}

{% step %}
Prepare payload with:

* Allocator `address`
* `Credit line (ceiling, dailyCap)`
* `borrowFeeBps` and `allowed=true`
* `assets[]` + `pocketsNew[]` (aligned in order)
{% endstep %}

{% step %}
Collect 4–9 EIP-712 signatures from `SIGNER` role addresses over the computed digest (includes `actionType`, `payloadHash`, `nonce`, `targetContract`).
{% endstep %}

{% step %}
Call `queueAction`; the governance contract verifies signatures and stores ETA.
{% endstep %}

{% step %}
After timelock, call `executeAction`; UCE processes `ACT_SET_ALLOCATOR`.
{% endstep %}

{% step %}
(Optional) Set or update pockets via `ACT_SET_ALLOCATOR_POCKETS`.
{% endstep %}

{% step %}
Allocator funds pockets, mints inventory, and starts serving referred flow.
{% endstep %}
{% endstepper %}
