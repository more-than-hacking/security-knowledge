# 06 — Response / Request Manipulation

## Client-side / response tampering
- **Flip the result:** intercept the verify response and change `{"success":false}` → `{"success":true}` / `status:200` — if the app trusts the client, you're in.
- This works when the **post-verify decision is made client-side** (mobile apps especially).

## Request-side bypass
- **Remove the OTP parameter** entirely — some endpoints treat missing as skip.
- **Empty / null / 0** OTP accepted (`otp=`, `otp=null`, `otp=0`).
- **Type juggling:** `otp` as array `otp[]=x`, object `{"otp":{}}`, boolean `true` — breaks loose comparisons (PHP `==`, NoSQL operators `{"$ne":null}`).
- **Default/backdoor codes:** `000000`, `123456`.
- **Parameter pollution:** send `otp` twice with a correct-looking value.

## Fix
- All decisions server-side; strict typing + strict comparison; reject missing/empty; treat any client-provided verification state as untrusted.
