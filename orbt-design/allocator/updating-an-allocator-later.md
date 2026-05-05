# Updating an Allocator later

Governance can amend parameters safely via the same action machinery:

* Pause/disable an allocator: ACT\_SET\_ALLOCATOR with op = UPDATE\_ALLOWED and allowed = false.
* Resize credit line: op = UPDATE\_LINE with new ceiling / dailyCap. (Ceiling cannot be set below current effective debt; the call reverts if so.)
* Adjust borrow fee: op = UPDATE\_BORROW\_FEE with new borrowFeeBps.
* Rotate referral code: op = UPDATE\_REFERRAL\_CODE triggers regeneration and mapping of a new code (old mapping cleared).
* Pocket changes: op = UPDATE\_POCKET or the single-asset ACT\_SET\_ALLOCATOR\_POCKETS.

Each update must be signed, queued, and timelocked before execution, just like onboarding.
