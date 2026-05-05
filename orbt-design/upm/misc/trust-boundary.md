# Trust boundary

User's Wallet

* Holds private keys
* Initiates transactions
* Grants approvals explicitly

\[User grants approval: WBTC → UPM] ▼

User Position Manager (UPM)

* Can spend user's WBTC (up to approved amount)
* Cannot withdraw without user transaction
* Transparent, immutable code

\[UPM executes calls to protocols] ▼

Protocol Contracts (UCE, USM, etc)

* Receive tokens from UPM (via transferFrom)
* Execute business logic
* Return results/tokens to specified recipients
