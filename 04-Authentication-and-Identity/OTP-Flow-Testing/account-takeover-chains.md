# 10 — Account Takeover Chains

OTP bugs are rarely the finding on their own — chain them to ATO for impact.

## Password reset via OTP
`/reset/send` (victim identifier) → brute/leak/replay the OTP → `/reset/complete` → set attacker password → **full ATO**.
- Common enablers: no attempt cap, OTP leakage in response, reset step reachable without OTP.

## Registration / verification bypass → pre-ATO
Register with the victim's email/phone, bypass verification (skip step / brute) → account created/verified under attacker control before the victim signs up.

## Email / phone change → ATO
Change email/phone with a weak OTP (no re-auth, brute-able, recipient tampering) → then reset password to the new attacker address.

## Disable-2FA → ATO
Turn off the victim's 2FA without a fresh OTP, then take over via password reset.

## Impact framing (for the report/interview)
State the **chain + business impact**: "unauthenticated attacker → brute a 6-digit reset OTP (no attempt cap) → full account takeover of any user by phone number."

## Fix
- Every step above needs: atomic attempt cap, single-use short-lived OTP, server-side binding to user+action, re-auth for sensitive changes, no leakage.
