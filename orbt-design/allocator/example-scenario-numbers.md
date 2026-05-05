# Example scenario (numbers)

{% stepper %}
{% step %}
Governance onboards Allocator A with:

* `ceiling = 10,000,000 0xUSD`, `dailyCap = 1,000,000`, `borrowFeeBps = 500` (policy-aligned),
* `assets = [USDC, DAI]`, `pockets = [P_USDC_A, P_DAI_A]`, `allowed = true`.
{% endstep %}

{% step %}
Allocator A mints $750k 0xUSD inventory via `allocatorCreditMint(A, 750_000)` → now has `reservedZeroX = 750k`; effective debt increases accordingly.
{% endstep %}

{% step %}
A user mints 0xUSD via USDC with A’s referral code:

* UCE retains liquid buffer (subject to change) on-hand (reserve), routes remaining to `P_USDC_A`.
* If the user instead redeems 0xUSD to USDC with A’s referral code, UCE burns their 0xUSD and consumes A’s `reservedZeroX` first.
{% endstep %}

{% step %}
A repays $300k using DAI: calls `allocatorRepay(DAI, 300_000e18)`.

* Contract skims borrow fee to treasury (per `borrowFeeBps`) and retires the 0x-equivalent of the principal from A’s debt.
{% endstep %}
{% endstepper %}
