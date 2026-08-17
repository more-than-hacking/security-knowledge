# 07 — Race Conditions in OTP Flows

## Where races bite
- **Attempt-limit overrun:** the "max 5 tries" counter is check-then-increment (non-atomic). Fire many verify requests **concurrently** (single-packet attack / Turbo Intruder) so they all read the counter before it updates → far more than 5 guesses.
- **Concurrent send:** multiple `/otp/send` at once → multiple valid OTPs, wider window, or counter resets.
- **Verify + resend race:** interleave to keep the attempt counter low while brute-forcing.

## How to test
- Turbo Intruder single-packet, or Burp "send group in parallel", with many candidate codes.

## Fix
- Atomic counter (DB `UPDATE ... WHERE attempts < N`, row lock, or atomic increment); invalidate OTP once cap hit; idempotency on send.
