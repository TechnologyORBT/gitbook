# Updating an Allocator later

Governance can amend parameters safely via the same action machinery:

* Pause/disable an allocator: `ACT_SET_ALLOCATOR` with `op = UPDATE_ALLOWED` and `allowed = false`.
* Resize credit line: `op = UPDATE_LINE` with new ceiling / dailyCap. (Ceiling cannot be set below current effective debt; the call reverts if so.)
* Adjust borrow fee: `op = UPDATE_BORROW_FEE` with new borrowFeeBps.
* Rotate referral code: `op = UPDATE_REFERRAL_CODE` triggers regeneration and mapping of a new code (old mapping cleared).
* Pocket changes: `op = UPDATE_POCKET` or the single-asset `ACT_SET_ALLOCATOR_POCKETS`.

Each update must be signed, queued, and timelocked before execution, just like onboarding.
