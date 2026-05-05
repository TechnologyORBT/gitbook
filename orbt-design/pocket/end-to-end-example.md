# End-to-End example

{% stepper %}
{% step %}
### Large mint flows to pocket

User swaps **$1,000,000 USDC → 0xUSD**.

* Example: UCE keeps **$250k** and sends **$750k** to pocket.
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
