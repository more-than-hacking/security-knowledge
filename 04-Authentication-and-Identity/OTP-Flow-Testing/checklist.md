# OTP Flow — One-Page Test Checklist

## Generation
- [ ] Length / entropy sufficient (≥6, CSPRNG)
- [ ] Not predictable / sequential / time-seeded
- [ ] No static/backdoor codes (000000, 123456)
- [ ] Same code not reused across requests/accounts

## Validation / expiry / binding
- [ ] Validated server-side (not a client flag)
- [ ] Single-use (invalidated after success)
- [ ] Expires (short TTL) and rejected after expiry
- [ ] Old OTP invalidated when a new one is requested
- [ ] Bound to user + session + action + identifier
- [ ] OTP for user A does NOT validate user B

## Brute force / rate limiting
- [ ] Atomic per-OTP attempt cap (3–5), then invalidate
- [ ] Per-account AND per-IP/device limits
- [ ] No bypass via X-Forwarded-For / headers / casing / new session
- [ ] Counter not reset by requesting a new OTP
- [ ] Consistent limits across all endpoints (web/mobile/GraphQL)
- [ ] Resend cooldown + daily cap (no SMS bombing)

## Reuse / replay
- [ ] Used/expired OTP cannot be replayed
- [ ] No cross-session / cross-action replay
- [ ] Post-verify token single-use + short-lived

## Leakage
- [ ] OTP not in response body/headers/URL/logs
- [ ] No status/poll endpoint reflecting the OTP
- [ ] No IDOR on OTP/verification object
- [ ] Uniform responses (no user/code oracle, no timing leak)

## Response / request manipulation
- [ ] success:false→true doesn't grant access
- [ ] Missing/empty/null/0 OTP rejected
- [ ] Array/object/type-juggling rejected
- [ ] Parameter pollution rejected

## Race conditions
- [ ] Concurrent verify can't overrun attempt cap
- [ ] Concurrent send doesn't widen window / reset counter

## Flow / logic
- [ ] Post-verify endpoint unreachable without proven OTP
- [ ] No force-browse / step reorder bypass
- [ ] Disable-2FA / email-change require fresh OTP + re-auth

## Delivery
- [ ] Recipient bound server-side (no destination tampering)
- [ ] Magic links use canonical host (no host-header poisoning)
- [ ] SMS treated as weakest; TOTP/push preferred; enrollment protected

## Chain to impact
- [ ] Attempted password-reset / verification / change → ATO chain
- [ ] Impact documented with the full chain
