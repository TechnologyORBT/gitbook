# Mechanics

* **Supply policy:**
  * Deposit all non-reserved USDC to Aave → receive **aUSDC**.
  * Optionally keep a small **on-pocket hot buffer** (e.g., 1–3% of pocket balance) to reduce borrow/withdraw round-trips for micro-settlements.
* **Withdrawal policy:**
  * For standard UCE [redemptions](../../uce/concepts/swaps.md#id-0x-a-redemption-side), rely on **allowance pulls** (pocket → UCE).
  * If the pocket’s on-pocket buffer is used, **rebalance** by withdrawing from Aave back to pocket and/or refilling UCE reserve.
* **Credit delegation policy:**
  * Maintain **per-delegate allowances** on Aave variable-debt tokens: `approveDelegation(delegate, limit)`.
  * Enforce **per-delegate ceilings**, **tenor limits** (time-boxed usage), and **global utilization caps** (e.g., “total delegated notional ≤ 25% of borrowable; target used ≤ 10%”).
  * Continuous **HF guardrails**: compute borrowable vs. delegated usage so **HF never < 2.0** under normal volatility.
* **Chain specifics:** Use per-chain Aave markets where liquidity is deep and risk params are conservative. Enable **eMode** for stable-stable if materially helpful and safe.
