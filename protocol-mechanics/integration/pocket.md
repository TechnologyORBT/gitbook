# Pocket

**Purpose:** The [**Pocket**](../../orbt-design/pocket/) is an allocator-controlled multisig EOA that : <br>

* Receives user-attributed inflows from [UCE](../../orbt-design/uce/) via referrals.
* Grants UCE pull-allowances for redemptions.
* Deploys idle balances into strategies via the [**User Position Manager (UPM)**](../../orbt-design/upm/).
  * Pockets earn base money-market yield and may optionally act as **On-Chain Clearing Houses (OCHs)** by _delegating credit_ to partnered settlement actors to capture intent fees.

***

### 1) Roles, Rights & Responsibilities

* **Custody:** Pocket directly holds underlying assets (USDC/USDT/DAI/…); it is a whitelisted allocator address.
* **Attribution:**[ Swaps](../../orbt-design/uce/concepts/swaps.md#swap-planes-pair-gated-conversion) with your referral code route inflows to **your Pocket** and consume **your `reservedZeroX`** first.
* **Settlement Readiness:** Keep ERC-20 allowances from Pocket → UCE sufficiently high for fast redemptions.
* **Deployment:** [Pocket](../../orbt-design/pocket/) is the _only caller_ authorized to instruct the [**UPM**](../../orbt-design/upm/) to operate strategies on its behalf.
* **Profit Sharing:** Strategy fees (bps on realized profit) are sent to **ORBT Treasury**; allocators earn the net yield + any OCH intent fees.

***

### 2) Control Plane: UPM (User Position Manager)

UPM brokers all strategy actions for Pockets and keeps ORBT strategy integrations **extensible**.

**UPM interface (Pocket-only entry points):**

```solidity
function doCall(address target, bytes data) external onlyRole(POCKET) returns (bytes result);
function doBatchCalls(address[] targets, bytes[] datas) external onlyRole(POCKET) returns (bytes[] results);
function doDelegateCall(address target, bytes data) external onlyRole(POCKET) returns (bytes result);
```

**Setup checklist**

1. Governance grants **POCKET** role on UPM to your Pocket address.
2. ORBT strategy contracts whitelist your Pocket (governance on strategy).
3. Pocket sets **sane allowances**:
   * To **UCE** (for redemptions/pulls).
   * To **strategy** (for aToken transfers on withdraw paths).

***

### 3) Two Deployment Modes

#### A) **Supply-Only (Money Market Yield)**

* Pocket supplies idle funds into ORBT’s [**Money Market Strategy**](../../orbt-design/pocket/aave-only-pocket-strategy/) **(Aave v3 adapter)** through UPM.
* Earns baseline deposit yield; liquidity stays withdrawable on demand for settlements.

**High-level call**

```
Pocket → UPM.doCall(OrbtMMStrategy, encodeWithSelector(supply(aToken, pocket, amount)))
```

#### Default Money-Market Supply Entrypoint (Pocket → UPM → OrbtMMStrategy)

**Supply function to call (default strategy):**

* **`OrbtMMStrategy.supply(address aToken, address pocket, uint256 amount)`**
* Must be invoked **via UPM** so that `onlyUPM` passes on the strategy.

**Call pattern (Pocket is the caller):**

```solidity
// from Pocket (multisig/EOA), calling UPM
bytes memory data = abi.encodeWithSelector(
    OrbtMMStrategy.supply.selector,
    aToken,              // Aave aToken for the underlying you’re supplying (e.g., aUSDC)
    pocket,              // your Pocket address
    amount               // underlying amount to supply (native decimals)
);
OrbtUPM(upm).doCall(address(orbtMMStrategy), data);
```

**Pre-requisites (one-time / per-asset):**

* **Whitelist:** Governance has whitelisted your `pocket` on the strategy (`isPocketWhitelisted[pocket] = true`).
* **UPM Role:** Governance granted your `pocket` the `POCKET` role on `UPM`.
* **Allowances:**
  * Pocket → **OrbtMMStrategy**: `IERC20(UNDERLYING).approve(orbtMMStrategy, amount)`\
    (Strategy pulls the underlying from the Pocket before depositing.)
* **aToken selection:** Use the **aToken** that corresponds to the underlying you’re supplying; the strategy infers:
  * `IAaveToken(aToken).UNDERLYING_ASSET_ADDRESS()` → underlying ERC-20
  * `IAaveToken(aToken).POOL()` → Aave v3 pool

**What the strategy does:**

1. Pulls `amount` of **underlying** from your `pocket`.
2. Deposits to Aave v3 on behalf of `pocket` (`ILendingPool.deposit(...)`).
3. Increments internal principal ledger for `pocket`/`aToken`.
4. Emits `Supplied(pocket, amount)`.

**Related exits & OCH hooks (for completeness):**

* Withdraw underlying back to the Pocket:\
  `withdrawFromPocket(aToken, pocket, amount, to)` or `withdrawAllFromPocket(aToken, pocket, to)` (via UPM).
* Delegate credit (OCH flow):\
  `approveDelegationFromPocketWithSig(...)` or `delegationFromPocketWithSig(...)` (via UPM).

#### B) **OCH (On-Chain Clearing House) via Credit Delegation**

* Pocket supplies to Aave **and** **delegates variable debt** to a partnered intent solver.
* Solver borrows against Pocket’s credit line to settle intents instantly; repays shortly after from inflows.
* Pocket earns **intent settlement fees** + deposit APY (minus strategy fee on profit).

**High-level calls**

```
Pocket → UPM.doCall(OrbtMMStrategy, encodeWithSelector(approveDelegationFromPocketWithSig(...)))
// or delegationFromPocketWithSig(...)
```

(Delegation details on the [Money Market](money-market-strategy.md).)

{% hint style="warning" %}
**Risk note:** Pocket bears borrower default risk up to delegated limit. Set delegation caps conservatively, monitor health factors and utilization.
{% endhint %}

***

### 4) Operational Checklist (Pocket)

* **Referrals:** Distribute your referral code; maintain **`reservedZeroX`** to serve your users instantly.
* **Allowances:** Keep UCE pull-allowances high; avoid redemption failures.
* **Liquidity Split:** Respect UCE **reserveBps** on inbound (held on UCE); rest flows to Pocket.
* **Strategy Usage:** Use UPM to **supply**, **withdraw**, and (optionally) **delegate credit**.
* **Monitoring:** Track aToken balances, principal, fee bps, outstanding delegations, and UCE reserve/utilization metrics.
