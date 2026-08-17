# 08 — Flow / Logic Bypass

## What to test
- **Skip the OTP step:** call the **post-verify endpoint directly** (e.g. `/reset/complete`, `/login/finalize`, `/txn/confirm`) without ever verifying.
- **Force-browse / step reordering:** jump to a later step; reorder multi-step requests.
- **State not enforced:** the server issues the session/reset purely on reaching the endpoint, not on proven OTP success.
- **Disable 2FA without OTP:** the "turn off 2FA" action doesn't require a fresh OTP.
- **Change flow confusion:** verify OTP for phone/email A but the action commits on account B (broken binding at the action step).
- **OTP required only on first factor:** password step rate-limited but OTP step not, or vice versa.

## Fix
- Server-side state machine: the post-verify action must require a server-verified, single-use, short-lived proof that THIS user completed THIS OTP for THIS action.
