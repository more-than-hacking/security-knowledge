# OTP Flow Testing

> Complete, self-contained guide to testing One-Time-Password (OTP) flows — login 2FA, password reset, phone/email verification, transaction confirmation, and step-up auth. Attacks, methodology, checklist, and fixes.

---

## 1. What is an OTP flow?

A short-lived secret (usually a 4–8 digit code, sometimes a magic link/token) sent over a side channel (SMS, email, authenticator app, push) to prove the user controls an identifier (phone/email) or to add a second factor.

Typical uses:
- **Login 2FA / MFA**
- **Password reset**
- **Phone / email verification** (signup, profile change)
- **Transaction / payment confirmation** (step-up auth)
- **Sensitive-action confirmation** (disable 2FA, change email)

## 2. The flow (map this before testing)

```
[1] REQUEST OTP        POST /otp/send        {identifier: phone/email}
        │  server generates OTP, stores (hash + expiry + attempts), sends via channel
        ▼
[2] DELIVERY           SMS / Email / App / Push
        ▼
[3] VERIFY OTP         POST /otp/verify      {identifier, otp}
        │  server compares, checks expiry + attempts, marks used
        ▼
[4] POST-VERIFY ACTION  issue session / reset password / confirm txn / enable 2FA
```

**Every arrow is a test surface.** Most real bugs are at [1] (request abuse), [3] (verify weaknesses), and [4] (skipping the step entirely).

## 3. Threat model — what an attacker wants

| Goal | Typical primitive |
|------|-------------------|
| Guess/brute the OTP | weak generation + no rate limit |
| Reuse/replay an OTP | not single-use / no expiry |
| Read the OTP directly | leakage in response/logs/IDOR |
| Skip OTP verification | flow/logic bypass, response tampering |
| Redirect OTP to attacker | identifier tampering, host-header |
| Account takeover | password-reset / email-change chains |

## 4. Root causes (say these in an interview)

- **Weak generation** — short length, low entropy, not a CSPRNG, predictable seed/timestamp.
- **No/weak rate limiting** — a 6-digit code (10^6 space) is trivially brute-forced without a hard, atomic attempt cap.
- **Not single-use / no expiry** — replay window.
- **Broken binding** — OTP not tied to the specific user/session/action, so an OTP for A validates B.
- **Missing server-side enforcement** — client-side "verified" flag, or the post-verify endpoint reachable without the verify step.
- **Leakage** — OTP returned in the response, logs, or fetchable via IDOR.

## 5. Test categories (files in this folder)

| # | File | What it covers |
|---|------|----------------|
| 01 | [otp-generation.md](otp-generation.md) | Predictability, entropy, length, reuse of same code |
| 02 | [otp-validation-and-expiry.md](otp-validation-and-expiry.md) | Single-use, expiry, binding, server-side enforcement |
| 03 | [rate-limiting-and-bruteforce.md](rate-limiting-and-bruteforce.md) | Brute force + every rate-limit bypass |
| 04 | [otp-reuse-replay.md](otp-reuse-replay.md) | Old/used OTP still valid, cross-session reuse |
| 05 | [otp-leakage.md](otp-leakage.md) | Response body/headers/URL/logs/IDOR disclosure |
| 06 | [response-manipulation.md](response-manipulation.md) | success:false→true, null/empty/array/type juggling |
| 07 | [race-conditions.md](race-conditions.md) | Concurrent verify (attempt-limit overrun), concurrent send |
| 08 | [flow-logic-bypass.md](flow-logic-bypass.md) | Skip the step, force-browse, disable-2FA without OTP |
| 09 | [delivery-channel-attacks.md](delivery-channel-attacks.md) | Recipient tampering, SMS bombing, SIM swap, email chain |
| 10 | [account-takeover-chains.md](account-takeover-chains.md) | Full ATO chains via reset / verification / change flows |
| — | [checklist.md](checklist.md) | One-page pre-engagement checklist |

## 6. Methodology

1. **Map** the 4-stage flow in Burp; note the exact request/response for send and verify.
2. **Generation** — request several OTPs (different accounts/times); look for sequential/predictable/repeated values (if ever leaked) and short length.
3. **Verify weaknesses** — brute force, reuse, expiry, binding, response tampering.
4. **Request abuse** — resend flooding, SMS bombing, no cooldown.
5. **Rate-limit bypass** — headers, IP rotation, casing, race, per-account vs per-IP gaps.
6. **Flow bypass** — try the post-verify endpoint directly; skip/force-browse.
7. **Leakage** — inspect every response, header, redirect, and any status endpoint; try IDOR on the OTP/status object.
8. **Chain** — combine into password reset / email change → account takeover; validate impact.

## 7. Tools

- **Burp Repeater** — manual verify/tamper tests.
- **Burp Intruder / Turbo Intruder** — brute force (000000–999999) and **single-packet race** for attempt-limit overrun.
- **Collaborator** — OOB leakage.
- **ffuf** — request/endpoint fuzzing.

## 8. Fixes / secure design (defense)

- Generate with a **CSPRNG**; **≥6 digits** (prefer longer or alphanumeric for high-value); never predictable/time-seeded.
- **Short expiry** (30–120s) and **single-use** — invalidate on success, on expiry, and on a new request.
- **Hard, atomic attempt cap** per OTP (e.g. 3–5) — increment atomically to survive races; then invalidate the OTP and require a new one.
- **Rate limit both per-account and per-IP/device**, with exponential backoff + lockout; **CAPTCHA** after N attempts.
- **Bind** the OTP to the exact user + session + action + identifier; verify server-side.
- **Never** return the OTP in responses/logs; no OTP in URLs; constant-time compare.
- **Limit resends** (cooldown + daily cap) to stop SMS bombing / cost abuse.
- **Enforce the step server-side** — the post-verify action must require proof the verify step succeeded (server-side state / signed short-lived token), not a client flag.
- Prefer **TOTP/authenticator or push** over SMS where possible; treat SMS as weakest (SIM swap).

## 9. Interview questions

- *"How would you test an OTP login flow end-to-end?"* → walk the 4 stages + the category list.
- *"A 6-digit OTP — is that secure?"* → only with a hard atomic attempt cap + short expiry + rate limiting; otherwise 10^6 is brute-forceable.
- *"Rate limiting vs the OTP attempt cap — difference?"* → rate limit = request volume; attempt cap = tries per specific OTP, enforced atomically (race-safe).
- *"How do OTP bugs become account takeover?"* → reset/verification/change flows + brute/leak/bypass.
- *"Why is SMS OTP the weakest factor?"* → SIM swap, SS7, delivery to attacker via number tampering.

## 10. What I initially misunderstood
_(fill as you learn — e.g. "rate limiting alone stops brute force" → false without an atomic per-OTP attempt cap; concurrency defeats naive counters.)_
