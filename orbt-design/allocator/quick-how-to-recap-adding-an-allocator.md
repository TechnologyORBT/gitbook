# Quick “how-to” recap: adding an Allocator

{% stepper %}
{% step %}
(One-time) Register UCE as governed (REGISTER\_CONTRACT).
{% endstep %}

{% step %}
(One-time) Register action types for UCE (REGISTER\_ACTION\_TYPE for ACT\_SET\_ALLOCATOR and ACT\_SET\_ALLOCATOR\_POCKETS).
{% endstep %}

{% step %}
Prepare payload with:

* Allocator address
* Credit line (ceiling, dailyCap)
* borrowFeeBps and allowed=true
* assets\[] + pocketsNew\[] (aligned in order)
{% endstep %}

{% step %}
Collect 4–9 EIP-712 signatures from SIGNER role addresses over the computed digest (includes actionType, payloadHash, nonce, targetContract).
{% endstep %}

{% step %}
Call queueAction; the governance contract verifies signatures and stores ETA.
{% endstep %}

{% step %}
After timelock, call executeAction; UCE processes ACT\_SET\_ALLOCATOR.
{% endstep %}

{% step %}
(Optional) Set or update pockets via ACT\_SET\_ALLOCATOR\_POCKETS.
{% endstep %}

{% step %}
Allocator funds pockets, mints inventory, and starts serving referred flow.
{% endstep %}
{% endstepper %}
