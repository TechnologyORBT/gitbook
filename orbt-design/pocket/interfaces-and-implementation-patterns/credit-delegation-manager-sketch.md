# Credit-delegation manager (sketch)

#### Responsibilities:

* Maintain **per-delegate limits** and **expiries**; call Aave `approveDelegation` on variable-debt tokens.
* Track **active delegated debt** per delegate; enforce **circuit breakers**.
* Expose `revoke(delegate)` and `pause()`; emit detailed events for monitoring.
