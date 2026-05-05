# Outbound (0xAsset → stable)

{% stepper %}
{% step %}
### User burns 0xAsset

User sends 0xAsset to UCE; UCE **swaps/burns** it.
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
