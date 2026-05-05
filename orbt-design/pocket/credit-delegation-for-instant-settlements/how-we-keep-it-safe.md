# How we keep it safe

* **Hard caps:** Per-delegate and global caps based on pocket’s collateral. Start small (e.g., **5–15%** of borrowable), increase with performance.
* **Tenor controls:** Allowances with **time locks** and **auto-expiry** (e.g., 30–120 minutes) for rapid settlement loops.
* **Kill switches:** Instant **revoke** (set delegation to 0), per-delegate pause, and global pause.
* **HF discipline:** Model worst-case utilization spikes and price shocks; set caps to keep **HF > 2.0** even under stress.
* **Escrowed repayment flows:** Where feasible, structure the solver’s flow so repayments are **atomic** with settlement proceeds or enforced by **post-trade bonds**.
