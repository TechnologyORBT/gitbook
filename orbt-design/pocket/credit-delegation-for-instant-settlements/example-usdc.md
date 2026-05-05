# Example (USDC)

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
