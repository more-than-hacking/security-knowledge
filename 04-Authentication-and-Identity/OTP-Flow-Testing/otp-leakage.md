# 05 — OTP Leakage / Disclosure

## Where OTPs leak
- **Response body:** `/otp/send` returns the OTP (dev leftover) — check JSON, hidden fields, comments.
- **Response headers:** custom headers echoing the code.
- **URL / redirect / Referer:** magic-link or `?otp=` leaks via history, logs, Referer to third parties.
- **Logs / error messages:** verbose errors or debug pages showing the code.
- **Status/poll endpoint:** a `/otp/status` or resend endpoint that reflects the current OTP.
- **IDOR on the OTP object:** `GET /otp/{id}` or fetching another user's pending OTP/verification record.
- **Timing / oracle:** different response for "valid code, wrong user" vs "invalid code" narrows the guess.

## How to test
Inspect **every** response (send, resend, status, verify), all headers, redirects; try IDOR on any OTP/verification identifier; use Collaborator for OOB.

## Fix
- Never return OTP anywhere client-visible; no OTP in URLs; scrub logs; authorize any status/verification object; uniform responses.
