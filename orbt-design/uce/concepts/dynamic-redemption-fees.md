# Dynamic Redemption Fees

### **Mechanism & Settlement Path.**

For **0x → A** redemptions, UCE charges a **dynamic fee** that is computed _in 0x units_ and remitted to the treasury _in underlying_. The fee is evaluated at a **snapshot rate** taken just before execution so that **preview = execution**.&#x20;

Execution flow:

* Compute `feeRate = current (decayed) baseRedemptionRate`.
* `feeZeroX = zeroXIn * feeRate`; convert to underlying via decimals normalization; transfer fee to treasury.
* Deliver `netA = grossA − feeA` to the user from reserves/pockets (no oracle, no underlying mint).
* **After** settlement, update pressure via `_onRedemptionExecuted(zeroXIn)`.

This mechanism is **oracle-free** on the redemption leg and **does not** scale allocator debt; it purely throttles redemption intensity and compensates the treasury during stress.

***

**Rate Dynamics & Stress Behavior.**\
The fee evolves with two opposing forces:

1.  **Decay (relief over time):**\
    Between events, the base rate decays **multiplicatively per hour** using a fixed factor (≈ **0.995** per hour, i.e., \~**0.5% hourly decay** of the _rate itself_), bounded to at most 24 hourly steps in a single read. Formally:\
    $$rate_t = rate_{t−Δ} × (0.995)^ (hoursElapsed)$$

    This pulls the fee back toward **0%** after pressure subsides, without governance action.
2.  **Bump (pressure from redemptions):**\
    After each redemption, the system computes the **redeemed fraction of total 0X supply**:

    ```
    redeemedFrac = zeroXRedeemed / totalSupply
    newRate = min( MAX_REDEMPTION_RATE(=5%),  decayedRate + redeemedFrac )
    ```

    Larger, sudden redemptions produce larger upward bumps. Frequent smaller redemptions keep the rate elevated until decay has time to dominate.

**Intuition under scenarios.**

* _Calm markets:_ few redemptions → small bumps; decay quickly returns the rate \~0%.
* _Redemption surge:_ big or clustered redemptions → rate jumps toward the **5% cap**, raising the marginal cost to exit and preserving pocket/reserve depth.
* _Recovery:_ as redemptions slow, the multiplicative decay rapidly lowers the fee back toward zero, restoring near-par exits.
