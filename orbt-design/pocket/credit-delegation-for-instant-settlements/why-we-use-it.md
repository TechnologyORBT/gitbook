# Why we use it

* **Latency:** The delegate can borrow and settle **in one transaction**—ideal for **intent-based settlement networks** (e.g., RFQ/solver systems).
* **Operational simplicity:** No pre-move needed from pocket to UCE; no allowance races; **no socialized slippage**—the delegate gets funds at par from Aave’s pool.
* **Determinism:** Delegation amounts, expiries, and per-delegate policies are on-chain and auditable.
