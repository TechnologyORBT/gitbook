# Pricing and Quotes

Let P(asset) be the USD price from the oracle (with staleness/deviation guards).&#x20;

Let H(asset) ∈ \[0,1] be a governance haircut (0 = no credit; 1 = par). Quotes follow:

$$
amountOut = amountIn * (P(in) * H(in)) / (P(out) * H(out)) * 10^Δdecimals
$$

{% hint style="info" %}
Conservative rounding applies (round down on payouts in 0xUSD terms; round up on liabilities).
{% endhint %}

