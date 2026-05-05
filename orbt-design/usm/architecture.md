# Architecture

{% stepper %}
{% step %}
### Core Vault Layer (ERC-4626)

<figure><img src="../../.gitbook/assets/7 - USM Architecture - Core Vault Layer (ERC-4626).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Core ORBT staking layer (Rewards Distribution)

<figure><img src="../../.gitbook/assets/7 - Core ORBT Staking Layer (Rewards Distribution).png" alt=""><figcaption></figcaption></figure>


{% endstep %}

{% step %}
### Governance Integration Layer

The USM integrates with the ORBT governance system for secure rate management.

Supported governance actions:

* `ACT_SET_RATE`: Update the per-second interest rate
* `ACT_SET_REWARD_CONFIG`: Configure external reward token emissions
* `ACT_SET_REWARDS_ONLY_MODE`: Toggle between full accrual and rewards-only mode

&#x20;Governance interface that is implemented for executing each action respectively:

```solidity
interface IGovernedContract {
    function executeGovernanceAction(bytes32 actionType, bytes calldata payload) external returns (bool);
}
```
{% endstep %}
{% endstepper %}
