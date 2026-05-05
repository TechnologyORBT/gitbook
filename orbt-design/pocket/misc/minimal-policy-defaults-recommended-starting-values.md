# Minimal policy defaults (recommended starting values)

* reserveBps (UCE on-hand): **25%** for USDC/USDT/DAI.
* Pocket hot buffer: **2%** of pocket balance.
* Delegation: global cap **≤ 15%** of borrowable; per-delegate **≤ 3%**; expiry **60 minutes**; HF floor **≥ 2.2** under worst-case stress.
* Alerts: UCE reserve < **10%** of 7-day p95 outflow; Aave utilization > **90%**; delegated usage > **50%** of cap; HF < **2.5** projected.
