# Money Market Strategy

**Contract:** `OrbtMMStrategy` (inherits `BaseStrategy`)

**Role of UPM:** Only UPM may call strategy entrypoints; Pockets are whitelisted allocators.

**Fee Model:** **`feeBps`** on _realized profit at withdrawal time_ → sent to ORBT Treasury. Principal is returned fee-free.

***

### 1) Key Interfaces (what integrators call via UPM)

**Supply & Withdraw**

```solidity
// Supply underlying from Pocket to Aave (aTokens accrue to Pocket)
function supply(address aToken, address pocket, uint256 amount) external onlyUPM;

// Withdraw underlying to `to`, pulling Pocket’s aTokens; strategy skims fee on profit portion
function withdrawFromPocket(address aToken, address pocket, uint256 amount, address to)
  external onlyUPM returns (uint256 withdrawn);

// Withdraw full aToken balance
function withdrawAllFromPocket(address aToken, address pocket, address to)
  external onlyUPM returns (uint256 withdrawn);
```

**Credit Delegation (OCH)**

```solidity
// Approve variable debt delegation (EIP-712 style) to `delegatee`
function approveDelegationFromPocketWithSig(
  address debtToken, address pocket, address delegatee, uint256 amount,
  uint256 deadline, uint8 v, bytes32 r, bytes32 s
) external onlyUPM;

// Alternative signature route used by some Aave deployments
function delegationFromPocketWithSig(
  address debtToken, address pocket, address delegatee, uint256 amount,
  uint256 deadline, uint8 v, bytes32 r, bytes32 s
) external onlyUPM;
```

**Debt Service Helpers**

```solidity
function repay(address aToken, address pocket, uint256 amount) external returns (uint256 repaid);
function repayWithPermit(address aToken, address pocket, uint256 amount,
  uint256 deadline, uint8 v, bytes32 r, bytes32 s) external returns (uint256 repaid);
```

**Read-Only**

```solidity
// Principal ledger per (pocket, aToken); used for PnL & fee computation
function principalOf(address aToken, address pocket) external view returns (uint256);

// BaseStrategy state (via events / governance):
// - feeBps, feeBpsOverrideByPocket[pocket], treasury, isPocketWhitelisted[pocket]
```

**Events**

```solidity
event Supplied(address indexed pocket, uint256 amount);
event Withdrawn(address indexed pocket, address indexed to, uint256 amount);
event Delegated(address indexed pocket, address indexed debtToken, address indexed delegatee, uint256 amount);
```

***

### 2) End-to-End Flows

#### A) Supply-Only Yield

**Approve & Supply**

1. Pocket approves **underlying → OrbtMMStrategy** for `amount`.
2. Pocket calls (via UPM):

```solidity
UPM.doCall(
  OrbtMMStrategy,
  abi.encodeWithSelector(OrbtMMStrategy.supply.selector, aToken, pocket, amount)
)
```

* Strategy pulls `amount` from Pocket, deposits to Aave on behalf of **Pocket**, mints aTokens **to Pocket**.
* Internal `principalByPocketAToken[pocket][aToken] += amount`.

**Withdraw (partial)**

1. Pocket approves **aToken → OrbtMMStrategy** for `amount`.
2. Pocket calls (via UPM):

```solidity
UPM.doCall(
  OrbtMMStrategy,
  abi.encodeWithSelector(OrbtMMStrategy.withdrawFromPocket.selector, aToken, pocket, amount, to)
)
```

* Strategy pulls Pocket’s aTokens, redeems underlying, computes **profit = amount - principalReduced**,\
  applies **feeBps** (or pocket override), sends **fee → Treasury**, **net → `to`**,\
  updates principal accordingly.

**Withdraw (all)**

* Same pattern using `withdrawAllFromPocket(...)`.

**Notes**

* Strategy enforces **no dust** post-ops; all temporary balances are cleared.
* Governance can set **global fee**, **per-pocket fee override**, treasury, whitelist.

***

#### B) OCH via Credit Delegation (Supply + Delegate + Earn Intent Fees)

**Concept**

* [Pocket](../../orbt-design/pocket/) supplies collateral (earning aToken yield).
* Pocket delegates variable debt limit on Aave to a **delegatee** (intent solver / clearing party).
* Delegatee borrows under Aave’s risk engine to fulfill intents instantly; later repays from settlement inflows.
* Pocket earns **intent settlement fees** (off-chain commercial terms) + deposit APY; ORBT takes fee only on realized _profit_ when Pocket unwinds.

**Delegate Flow**

1. Obtain `debtToken` (Aave variable debt token address for the asset).
2. Pocket signs a delegation approval (EIP-712).
3. Pocket calls (via UPM):

```solidity
UPM.doCall(
  OrbtMMStrategy,
  abi.encodeWithSelector(
    OrbtMMStrategy.approveDelegationFromPocketWithSig.selector,
    debtToken, pocket, delegatee, delegateLimit, deadline, v, r, s
  )
)
```

(or `delegationFromPocketWithSig` variant if required by deployment.)

**Risk Controls**

* Use **bounded `delegateLimit`**; monitor Aave health factor and outstanding variable debt.
* Revoke/resize delegation via a fresh signature with new amount when needed.
* Keep **aToken principal** sized to maintain healthy HF buffers.

**Debt Service**

* Anyone can help repay on behalf of Pocket:

```solidity
OrbtMMStrategy.repay(aToken, pocket, amount)        // needs allowance
OrbtMMStrategy.repayWithPermit(aToken, pocket, ...) // EIP-2612
```

***

### 3) Integration Checklists

**Pocket Runtime**

* Maintain **UCE** allowances for redemptions.
* Maintain **strategy** allowances:
  * underlying → `OrbtMMStrategy` (supply),
  * aToken → `OrbtMMStrategy` (withdraw).
* For OCH:
  * Supply collateral first; then sign and submit **credit delegation** via UPM.
  * Monitor HF, delegated limit utilization, and repayments.

**Monitoring**

* `principalOf(aToken, pocket)` for basis tracking.
* aToken balances, UCE reserve utilization, delegated amounts (via debtToken views), fee parameters (events/governance).
* Alerts on low allowances, high utilization, or stale governance settings.

***

### 4) Commercial Outcomes

* **Supply-Only:** Earn conservative, liquid Aave APY; fee applied only on realized profit at withdrawal.
* **OCH (Credit Delegation):** Stack **intent settlement fees** on top of APY by powering partnered solver networks. Highest expected yield; assumes disciplined risk limits and active monitoring.

**Bottom line:** Allocators can start with **Supply-Only** for simplicity, then graduate to **OCH** to maximize yield once processes and monitoring are in place.
