# 04 — OTP Reuse / Replay

## What to test
- **Used OTP replay:** verify once (success), then replay the exact request — still accepted?
- **Expired OTP replay:** replay after TTL.
- **Cross-session replay:** capture OTP in session A, use it in session B / another device.
- **Cross-action replay:** OTP issued for one action replayed for another.
- **Token after verify:** does verify return a token/flag that is itself replayable indefinitely to reach the post-verify action?

## Fix
- Strict single-use; invalidate on success/expiry/new-request; bind to session+action; make any post-verify token short-lived and single-use.
