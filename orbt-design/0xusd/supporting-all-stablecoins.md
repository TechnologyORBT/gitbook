# Supporting “All Stablecoins”

Governance whitelists assets with per-asset controls:

* **Onboarding tuple**: (asset, family, pocket, reserveBps). For 0xUSD we admit multiple USD stables and wrappers.
* **Haircuts and Pauses**: Quotes incorporate both P(asset) and H(asset). During a depeg, governance can reduce H(asset) (or pause the asset), preventing value leakage from 0xUSD holders into a depegged underlying.
* **Concentration and Caps**: Per-asset effective capacity prevents over-exposure. When a cap is hit, governance can slow routing by increasing reserveBps elsewhere or pausing new deposits.
