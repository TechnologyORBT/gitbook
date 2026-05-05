# Security best practices

For users:

* Minimal approvals: Approve only needed amounts
* Revoke unused approvals: Use tools like Revoke.cash
* Verify call data: Ensure batched calls do what you expect
* Use reputable frontends to construct call data
* Check transaction results after execution

For frontend developers:

* Simulate transactions (eth\_call) to preview outcomes
* Validate and sanitize user inputs before encoding
* Show users exact actions the tx will perform
* Provide meaningful error messages
* Include slippage protection in call data
