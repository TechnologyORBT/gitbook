# Governance and Policy Layer setup

The policy layer of ORBT enforces governance and role management through timelocked, multi-signature EIP-712 actions. All Allocator permissions, limits, and parameters are managed through this layer.

### Signers & thresholds

* Roles
  * ADMIN: manages signer sets, cancels queued actions.
  * SIGNER: addresses authorized to sign actions.
* Thresholds
  * A minimum of 4 and maximum of 9 signatures are enforced on every action (configurable by governance via the SET\_THRESHOLDS core action).
* Replay & duplication safety
  * Each action includes a nonce; the signed EIP-712 digest encodes actionType, payloadHash, nonce, and targetContract.
  * Governance verifies signatures are from distinct SIGNERs; duplicates are rejected.
  * Digests are consumed on execution to prevent replays.

### &#x20;Action types & routing

* Governance maintains a registry of valid action types (bytes32 identifiers).
  * Core action types (e.g., REGISTER\_ACTION\_TYPE, SET\_TIMELOCK, REGISTER\_CONTRACT) are always allowed.
  * Product actions (e.g., ACT\_SET\_ALLOCATOR, ACT\_SET\_ALLOCATOR\_POCKETS) must be registered first via REGISTER\_ACTION\_TYPE.
* After timelock, actions are executed:
  * Core actions run inside ORBTGovernance.
  * All others are routed to the target governed contract (e.g., Orbit UCE) which exposes executeGovernanceAction(actionType, payload).

#### Timelock <a href="#id-5.3-timelock" id="id-5.3-timelock"></a>

* Each queued action receives an ETA = now + actionTimeLock.
* Execution is only possible after ETA.
* Admins can cancel queued actions before ETA; cancellations and executions emit events for auditability.

<br>
