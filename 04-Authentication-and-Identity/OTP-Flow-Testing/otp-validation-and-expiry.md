# 02 — OTP Validation, Expiry & Binding

## What to test
- **Server-side validation:** is the check done server-side, or is there a client-side "verified" flag you can flip? (see 06 & 08)
- **Single-use:** after a successful verify, does the same OTP work again? It must be invalidated.
- **Expiry:** wait past the stated TTL — is it still accepted? Is there any expiry at all? Some OTPs stay valid for hours/forever.
- **Invalidated on new request:** requesting a fresh OTP should invalidate the old one; if both work, the valid window widens.
- **Binding:**
  - OTP generated for **user A** — does it validate **user B**? (broken user binding → mass ATO)
  - OTP for **action X** (e.g. login) accepted for **action Y** (e.g. disable 2FA)?
  - OTP not bound to session/device — captured OTP usable from another session.
- **Verification against the wrong identifier:** send `otp` of account A with `identifier` of account B.

## Fix
- Single-use + short expiry; invalidate old on new; bind OTP to {user, session, action, identifier}; constant-time compare.
