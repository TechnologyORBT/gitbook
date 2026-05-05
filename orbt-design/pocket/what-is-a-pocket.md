# What is a "pocket"

In Orbt UCE, a **pocket** is “where the money waits”, the custody endpoint for each whitelisted stablecoin:

* **Address type:** EOA, multisig, or a minimal strategy vault controlled by governance or allocator policy.
* **Accounting role:**
  * **On UCE:** a per-asset **reserve buffer** (`reserveBps`) is retained on-hand.
  * **In the pocket:** the remainder of inbound user funds is deposited to Aave (aTokens accrue interest).
* **Permissions:** Pockets **approve** UCE to spend (pull) up to an allowance; UCE calls `safeTransferFrom(pocket, UCE, amount)` during redemptions.
* **Routing:** For user mints or redeems with a [referral](../uce/concepts/allocators/referral-vs-non-referral.md), UCE routes inflows to (and pulls from) the **referrer’s pocket** first; otherwise it uses the **global pocket**.

In short: [**UCE**](../../protocol-mechanics/integration/uce.md) **handles quotes and burning/minting; pockets hold the underlyings and produce yield while staying permissioned for fast pulls.**
