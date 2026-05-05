# Reserve buffer & rebalancing

* [**On-hand reserves**](../uce/concepts/reserve-policy.md) **(UCE):** Parameterized by `reserveBps` (typ. 15–35% for core stables).
* [**Pocket buffer**](reserve-buffer-and-rebalancing.md) **(optional):** Keep a small **hot float** (1–3%) to avoid withdraw/borrow on tiny bursts.
* **Rebalancing rules:**
  * If UCE reserve falls below a watermark, **pull** from pocket (or **withdraw** from Aave to pocket, then pull).
  * If UCE reserve is above a ceiling, **push** excess to pocket, then **supply** to Aave.

<br>
