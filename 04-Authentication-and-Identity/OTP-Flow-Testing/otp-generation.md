# 01 — OTP Generation

## What to test
- **Length / space:** 4-digit = 10^4, 6-digit = 10^6. Short codes are brute-forceable if attempts aren't capped.
- **Predictability:** request many OTPs (across accounts/time). Sequential, incrementing, or repeating codes = weak PRNG or timestamp seed.
- **Same OTP reused:** does requesting again return the *same* code? Does the same code work for multiple accounts?
- **Entropy source:** not a CSPRNG (e.g. `rand()` seeded by time) → predictable.
- **Static / default OTP:** dev backdoors like `000000`, `123456`, `111111` accepted in prod.

## How to observe values
You usually can't see the OTP unless there's **leakage** (see 05) or you control the delivery channel (your own phone/email in a test account). Use your own accounts to collect real codes and look for patterns.

## Fix
- CSPRNG; ≥6 digits (longer/alphanumeric for high value); no predictable seed; no static/test codes in prod.
