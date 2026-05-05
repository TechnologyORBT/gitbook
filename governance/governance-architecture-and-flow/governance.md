# Governance

## Overview

The ORBT Governance Module is a sophisticated, multi-signature timelock governance system designed to provide secure, decentralized control over critical protocol operations. Built on EIP-712 typed structured data signing, the module enforces a queuing and timelock mechanism that prevents immediate execution of sensitive actions, providing transparency and allowing for community review before changes take effect.

The governance architecture separates action proposal (queuing) from execution, requiring M-of-N multisignature approvals and a mandatory time delay. This design pattern is inspired by industry-standard governance systems like Compound's Timelock and MakerDAO's governance contracts, but extends them with dynamic action type registration, flexible contract management, and replay protection.

Key features include:

* Multi-signature requirement: Configurable M-of-N signature thresholds (minimum and maximum)
* Timelock enforcement: Mandatory delay between queuing and executing actions
* Dynamic action registry: Ability to register new action types without contract upgrades
* Contract whitelisting: Only registered contracts can be governed
* Replay protection: Each action can only be executed once using cryptographic digest tracking
* EIP-712 compliance: Industry-standard typed data signing for off-chain signature generation
* Core action handling: Built-in support for threshold updates, timelock changes, and registry management

The module is designed to be the single source of truth for governance across all ORBT protocol contracts, eliminating code duplication and providing a unified governance interface.

***

## Architecture

The governance system consists of three primary components:

### 1. ORBTGovernance Contract

The central governance contract (ORBTGovernance.sol) that handles:

* Action queuing with signature verification
* Timelock enforcement
* Action execution routing
* Registry management (action types and contracts)
* Core governance operations (thresholds, timelock adjustments)

### 2. IGovernedContract Interface

Contracts that wish to be governed must implement the IGovernedContract interface:

```solidity
interface IGovernedContract {
    function executeGovernanceAction(bytes32 actionType, bytes calldata payload)
        external returns (bool);
}
```

This interface serves as the entry point for all governance actions. When an action is executed, the governance contract calls this function on the target contract, passing the action type and payload. The target contract then decodes the payload and executes the appropriate internal logic.

### 3. GovernanceIntegration Base Contract

A helper base contract (GovernanceIntegration.sol) that provides common functionality for governed contracts:

* Access control integration
* Governance reference management
* onlyGovernance modifier for restricting function access

***

## Concepts

### Roles

ADMIN Role

* Responsibilities:
  * Grant and revoke SIGNER roles
  * Cancel queued actions that are potentially malicious or erroneous
  * Manage initial setup and emergency interventions
* Powers:
  * Call grantRole(SIGNER, address) and revokeRole(SIGNER, address)
  * Call cancelAction() to remove actions from the queue
* Security: Should be held by a trusted multisig or DAO treasury
* Limitations: Cannot directly execute actions or bypass timelock

SIGNER Role

* Responsibilities:
  * Review and sign governance proposals off-chain
  * Provide cryptographic attestation for action approval
* Powers:
  * Signatures are aggregated and verified during queueAction()
  * No direct on-chain transaction privileges
* Security: Should be distributed across multiple independent parties
* Limitations: Cannot queue or execute actions independently; requires M-of-N cooperation

### Signer Thresholds

The governance system uses flexible M-of-N multisignature thresholds:

* Minimum Signatures (M)\
  The minimum number of valid SIGNER signatures required to queue an action. Example: if minSignatures = 3, at least 3 distinct signers must provide valid signatures.
* Maximum Signatures (N)\
  The maximum number of signatures that can be provided. This bounds verification gas cost and prevents signature spam. Example: if maxSignatures = 5, submitting 6 signatures will fail even if all are valid.

Dynamic Threshold Updates

Thresholds can be adjusted through governance itself via the SET\_THRESHOLDS action:

```solidity
// Payload encoding
bytes memory payload = abi.encode(newMinSignatures, newMaxSignatures);
```

Validation Rules:

* minSignatures must be ≥ 1
* maxSignatures must be ≥ minSignatures
* Changes take effect only after queuing, timelock, and execution

Use Cases:

* Increasing security: Raise minSignatures as protocol value grows
* Operational flexibility: Adjust thresholds based on signer availability
* Progressive decentralization: Start with lower thresholds, increase over time

***

## Queuing Actions

Queuing is the first phase of governance execution. It registers an action's intent and validates that sufficient signers approve the proposal.

{% stepper %}
{% step %}
### Off-Chain Coordination

* Signers agree on action details (actionType, payload, nonce, targetContract)
* Each signer computes the EIP-712 digest:

```solidity
bytes32 structHash = keccak256(abi.encode(
    ACTION_TYPEHASH,
    actionType,
    payloadHash,
    nonce,
    targetContract
));
bytes32 digest = keccak256(abi.encodePacked(
    "\x19\x01",
    DOMAIN_SEPARATOR,
    structHash
));
```

* Signers sign the digest using their private keys (e.g., via hardware wallets)
{% endstep %}

{% step %}
### On-Chain Queuing

* Anyone can call queueAction() with collected signatures.
* The contract verifies:
  * Action type is registered: validActionTypes\[actionType] == true
  * Target contract is registered (if not core action): registeredContracts\[target] == true
  * Action not already queued or executed: usedActionDigests\[digest] == false
  * Signature count ≥ minSignatures and ≤ maxSignatures
  * All signatures are valid and from distinct SIGNER addresses
* If valid, the action is scheduled for execution at block.timestamp + actionTimeLock
* ActionQueued event is emitted with digest, action type, target, ETA, and nonce
{% endstep %}

{% step %}
### Digest Calculation

The digest used as the unique identifier across queuing and execution:

```solidity
bytes32 digest = keccak256(abi.encode(
    actionType,
    payloadHash,
    nonce,
    targetContract
));
```
{% endstep %}

{% step %}
### Signature Verification

The internal verification performs these checks:

```solidity
function _verifySignatures(
    bytes32 digest,
    bytes[] calldata signatures
) internal view {
    // 1. Check signature count bounds
    require(signatures.length >= minSignatures, "Too few");
    require(signatures.length <= maxSignatures, "Too many");

    // 2. Recover each signature and verify signer
    address lastSigner = address(0);
    for (uint256 i = 0; i < signatures.length; i++) {
        address signer = ECDSA.recover(digest, signatures[i]);

        // 3. Verify signer has SIGNER role
        require(hasRole(SIGNER, signer), "Not signer");

        // 4. Prevent duplicate signatures (sorted check)
        require(signer > lastSigner, "Duplicate or unsorted");
        lastSigner = signer;
    }
}
```

Security considerations:

* Signatures must be sorted by signer address to detect duplicates in O(n) time
* ECDSA recovery is gas-intensive; maxSignatures prevents DoS
* Invalid signatures revert the entire transaction
* Signers cannot sign the same action twice
{% endstep %}
{% endstepper %}

***

## Timelock

The timelock is a critical safety mechanism that introduces a delay between action approval and execution. This delay provides:

* Transparency: Community review of pending actions
* Intervention window: Time to cancel malicious or erroneous actions
* Predictability: Actions execute at known future timestamps

### Timelock Enforcement

```solidity
function _enforceTimelockAndConsume(bytes32 digest) internal {
    uint256 eta = queuedActionEta[digest];

    // 1. Verify action is queued
    require(eta != 0, "Not queued");

    // 2. Verify timelock has elapsed
    require(block.timestamp >= eta, "Timelock not elapsed");

    // 3. Mark digest as used (replay protection)
    usedActionDigests[digest] = true;

    // 4. Clear queue entry
    delete queuedActionEta[digest];
}
```

Timelock Duration (actionTimeLock):

* Development/Testing: 0 seconds
* Testnet: 1 hour (3600 seconds)
* Mainnet: 24-48 hours (86400-172800 seconds)

Updating Timelock

The timelock can be updated via governance (SET\_TIMELOCK). Important: changing the timelock does NOT affect already-queued actions; they retain their original ETAs.

***

## Action Execution

Action execution is the final phase where approved and time-locked actions are applied to the protocol.

{% stepper %}
{% step %}
### Initiation

* Anyone can call executeAction() once timelock has elapsed.
* This permissionless design ensures actions cannot be censored by admins.
{% endstep %}

{% step %}
### Routing & Execution (high-level)

```solidity
function executeAction(
    bytes32 actionType,
    bytes calldata payload,
    uint256 nonce,
    address targetContract
) external returns (bool success) {
    // 1. Compute digest and enforce timelock
    bytes32 digest = computeDigest(actionType, payloadHash, nonce, targetContract);
    _enforceTimelockAndConsume(digest);

    // 2. Route to appropriate handler
    if (_isCoreAction(actionType)) {
        success = _executeCoreAction(actionType, payload);
    } else {
        success = IGovernedContract(targetContract)
            .executeGovernanceAction(actionType, payload);
    }

    // 3. Emit event
    emit ActionExecuted(digest, actionType, targetContract, success);
}
```
{% endstep %}

{% step %}
### Core Action Handling

Core actions handled internally by ORBTGovernance:

* SET\_THRESHOLDS: Updates minSignatures and maxSignatures
* SET\_TIMELOCK: Updates actionTimeLock
* REGISTER\_ACTION\_TYPE: Adds new action type to validActionTypes mapping
* UNREGISTER\_ACTION\_TYPE: Removes action type
* REGISTER\_CONTRACT: Adds contract to registeredContracts mapping
* UNREGISTER\_CONTRACT: Removes contract
{% endstep %}

{% step %}
### External Contract Execution

For non-core actions, ORBTGovernance delegates to the target contract:

```solidity
IGovernedContract(targetContract).executeGovernanceAction(actionType, payload);
```

The target contract (e.g., OrbitUCE, sOxAsset) decodes the payload and applies the changes.
{% endstep %}
{% endstepper %}

Execution Security

* Atomicity: Either the entire action succeeds or reverts; no partial execution
* Reentrancy Protection: ReentrancyGuard prevents reentrant calls during execution
* Gas Limits: Complex actions may require higher gas limits; test on testnet first
* Failure Handling: If execution fails, the action is still marked as used (cannot retry with same nonce)

***

## Replay Protection

Replay protection prevents the same governance action from being executed multiple times.

Mechanism:

1. Digest-Based Tracking

```solidity
mapping(bytes32 => bool) public usedActionDigests;
```

Once an action is executed, its digest is permanently marked as used:

```solidity
usedActionDigests[digest] = true;
```

2. Nonces

Each action includes a nonce parameter chosen by the signers. To replay the same logical action, signers must:

* Use a different nonce (changes the digest)
* Collect fresh signatures for the new digest
* Queue and wait for timelock again

Queue Protection

```solidity
require(queuedActionEta[digest] == 0, "Already queued");
require(!usedActionDigests[digest], "Already executed");
```

Nonce Strategy

* Sequential Nonces (Recommended)

```solidity
// Track last used nonce per action type
mapping(bytes32 => uint256) public nonces;

// Increment when queuing
uint256 nonce = nonces[actionType]++;
```

Benefits: Prevents accidental reuse, provides execution ordering, easy to audit.

* Random Nonces (Alternative)

```solidity
uint256 nonce = uint256(keccak256(abi.encode(block.timestamp, msg.sender)));
```

Benefits: No coordination needed between signers; slight collision risk (negligible with 256-bit space). Trade-offs: No ordering guarantees.

***

## Attack Prevention

* Malicious Replay Attempt: An attacker calling queueAction() with a digest already marked used will revert due to usedActionDigests\[digest] == true.
* Front-Running: Attempting to queue the same action before it executes will revert because queuedActionEta\[digest] != 0.

***

## Integration Guide — For Contract Developers

To make your contract governable by ORBTGovernance:

1. Implement IGovernedContract

```solidity
import {IGovernedContract} from "src/governance/IORBTGovernance.sol";

contract MyContract is IGovernedContract {
    IORBTGovernance public governance;

    modifier onlyGovernance() {
        require(msg.sender == address(governance), "Not governance");
        _;
    }

    function executeGovernanceAction(
        bytes32 actionType,
        bytes calldata payload
    ) external onlyGovernance returns (bool) {
        if (actionType == keccak256("MY_ACTION")) {
            (uint256 param1, address param2) = abi.decode(payload, (uint256, address));
            _handleMyAction(param1, param2);
            return true;
        }
        revert("Unknown action");
    }
}
```

2. Define Action Types

```solidity
bytes32 internal constant ACT_MY_ACTION = keccak256("MY_ACTION");
```

3. Implement Handlers

```solidity
function _handleMyAction(uint256 param1, address param2) internal {
    // Apply changes with validation
    require(param1 > 0, "Invalid param");
    myState = param1;
    emit MyActionExecuted(param1, param2);
}
```

***

## Security Considerations

* Signer Key Management: Use hardware wallets and secure key storage
* Timelock Duration: Longer timelocks provide more security but reduce agility
* Signature Ordering: Always sort signatures by signer address
* Gas Limits: Complex payloads may require higher gas limits
* Emergency Actions: Consider a separate emergency role for critical interventions
* Audit: Have governance logic professionally audited
* Testing: Comprehensive unit and integration tests for all action types

{% hint style="warning" %}
Failure during execution still consumes the action (marks digest as used). Design actions and tests accordingly.
{% endhint %}

***

## Example Usage

1. Register an action type

```solidity
bytes32 actionType = keccak256("SET_ALLOCATOR");
bytes memory payload = abi.encode(actionType, "Set allocator configuration");

// ... queue and execute via governance
```

2. Execute a governed action

```solidity
bytes memory payload = abi.encode(allocatorAddress, allowed, ceiling, dailyCap);
bytes32 payloadHash = keccak256(payload);
uint256 nonce = 1;
address target = address(orbitUCE);

// Off-chain: Collect signatures from signers
bytes[] memory signatures = collectSignatures(actionType, payloadHash, nonce, target);

// On-chain: Queue
governance.queueAction(actionType, payloadHash, nonce, target, signatures);

// Wait for timelock...
vm.warp(block.timestamp + governance.actionTimeLock());

// Execute
governance.executeAction(actionType, payload, nonce, target);
```

***

If you want, I can:

* Convert specific code examples into titled code blocks for GitBook's code block component,
* Add quick reference tables for core actionType constants and their expected payload encodings,
* Or extract the developer integration snippets into a standalone "Governance Integration" page. Which would you prefer?
