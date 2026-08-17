# 09 — Delivery Channel Attacks

## Recipient tampering (most important for app testing)
- **Change the destination:** tamper `phone`/`email` in `/otp/send` (or a hidden field) so the OTP is delivered to an **attacker-controlled** recipient while the action targets the victim.
- **Host header / link poisoning:** magic-link OTP built from the `Host`/`X-Forwarded-Host` header → attacker-controlled link.
- **Second recipient / CC:** parameters that add a recipient.

## Channel-specific
- **SMS:** SIM swap, SS7 interception, SMS bombing (no resend cap). Treat SMS as the weakest factor.
- **Email:** if the email account is compromised or the email-change flow is weak, OTP → ATO. Email OTP inherits email account security.
- **Authenticator (TOTP):** stronger; test enrollment (can attacker enroll their own?), backup codes, and clock-window abuse.
- **Push:** MFA fatigue / prompt bombing; ensure number-matching.

## Fix
- Bind recipient server-side to the account (never trust client-supplied destination for an existing user); canonical host for links; resend caps; prefer TOTP/push with number-matching; protect enrollment.
