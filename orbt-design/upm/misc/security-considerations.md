# Security Considerations

Audit status

* Status: \[Pending/Completed]
* Auditor: \[Firm Name]
* Report: \[Link to audit report]

Known risks

1. Approval risk — granting max approval to UPM is a trust decision
2. Call data risk — malicious call data could drain tokens if frontend is untrusted
3. Reentrancy — UPM itself has no reentrancy guards; targets must be safe
4. Flash loan attacks — UPM can be used to facilitate attacks elsewhere
5. Gas limits — very large batches may hit block gas limits

Mitigation strategies

* Protocol level: minimal audited code, use Address.functionCall (OpenZeppelin), immutable deployment
* User level: use reputable frontends, review txs, manage approvals
* Frontend level: simulate txs, preview actions, slippage protection, validate inputs

Emergency response

1. Community alert (Discord/Twitter/forum)
2. Revoke approvals
3. Deploy patched UPM (new address)
4. Coordinate migration
