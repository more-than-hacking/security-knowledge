# 03 — Rate Limiting & Brute Force

## The core bug
A 6-digit OTP is only 1,000,000 values. Without a **hard, atomic attempt cap per OTP**, you can brute-force it. Rate limiting on *requests* is NOT the same as capping *tries per OTP*.

## Brute force
- Burp Intruder / Turbo Intruder over `000000`–`999999` on `/otp/verify`.
- Watch for: no cap, a high cap (e.g. 100s), or a cap that **resets** when you request a new OTP (request-new + keep brute-forcing).

## Rate-limit bypass techniques
- **Header spoofing:** `X-Forwarded-For`, `X-Real-IP`, `X-Client-IP`, `True-Client-IP` rotation to reset per-IP limits.
- **Per-account vs per-IP gap:** limit is per-IP but not per-account (or vice versa) — rotate the missing dimension.
- **Case / format tricks:** change casing, add spaces, leading zeros, `+`/country-code variants of the identifier.
- **New session / token per request:** if the limiter keys on session.
- **Race conditions:** fire many verify requests concurrently to overrun a non-atomic counter (see 07).
- **Reset counter via resend:** requesting a new OTP resets the attempt counter → brute in batches.
- **Endpoint variants:** `/verify`, `/v2/verify`, mobile API, GraphQL mutation — limits applied inconsistently.

## Resend / request-side abuse
- No cooldown/cap on `/otp/send` → **SMS bombing** (spam victim, cost/DoS) and OTP-window widening.

## Fix
- Atomic per-OTP attempt cap (3–5) → then invalidate; per-account AND per-IP/device limits; exponential backoff + lockout; CAPTCHA after N; resend cooldown + daily cap; only trust proxy headers from your own edge.
