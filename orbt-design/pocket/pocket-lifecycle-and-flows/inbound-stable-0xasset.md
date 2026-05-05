# Inbound (stable → 0xAsset)

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
