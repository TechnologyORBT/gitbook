# Reserve Policy

UCE maintains a **per-asset reserve** to guarantee fast, deterministic settlement for **OX → U** redemptions without waiting on vault (pocket) withdrawals. For each supported underlying, governance sets a **`reserveBps`** (default **25%**) that determines what fraction of every inbound **U → OX** deposit is **retained on the UCE contract** as immediately spendable liquidity; the remainder is forwarded to the asset’s configured **pocket** (custody address).

**How it works (lifecycle):**

* **On U → OX:** The engine splits the received underlying for that asset:\
  **reserve slice** stays on-contract; **pocket slice** is forwarded to the asset’s pocket (or the referral allocator’s pocket, when applicable). The reserve is tracked per asset and reflected on-chain in the contract’s balance, making capacity auditable.
* **On OX → U:** Redemption sources underlying in priority: **(1) on-hand reserve for that asset**, consuming tracked units first; **(2) referral allocator pocket** (if provided); **(3) global pocket**. All pocket pulls are strictly **bounded by allowance and balance**—no implicit rehypothecation, no synthetic underlying mint.
* **Per-asset tuning:** Each underlying has its own `reserveBps`, enabling heterogeneous liquidity postures (e.g., higher reserve for choppy assets, lower reserve for deep, stable assets). The **25% default** is a balanced posture; typical operating ranges are **10–50%** depending on redemption latency targets, pocket refill cadence, and yield trade-offs.

**Why it matters (trade-offs & guidance):**

* **Latency vs. yield:** Higher reserves improve **instant redemption depth** and reduce pocket dependency, at the cost of **lower capital efficiency** (less principal earning yield in pockets). Lower reserves maximize yield but increase reliance on pocket pulls and their gas/latency profile.
* **Transparency & solvency:** Because reserves are **on-contract balances** and pocket pulls are **allowance-bounded**, redemption capacity is **explicit and auditable**; insufficient pocket liquidity results in a **revert** rather than hidden shortfalls.
* **Interplay with dynamic fees:** During redemption surges, the **dynamic fee** raises marginal exit cost while the **reserve** buffers immediate demand. As pressure decays, fee recedes and reserves are replenished by new U inflows and pocket refills.

**Operational tips:**

* Increase `reserveBps` for assets with **volatile demand** or **slower pocket withdrawal SLAs**.
* Decrease `reserveBps` when **pocket liquidity is abundant**, withdrawal allowance is well-managed, and **yield** is the priority.
* Review reserves jointly with **tin** (mint fee) and **dynamic redemption fee** settings to maintain consistent peg behavior and treasury runway.
