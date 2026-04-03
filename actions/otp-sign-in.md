# OTP Sign-In

Authenticate a user via email OTP — no dashboard or browser required. Ideal for CLI tools, desktop apps, and AI agents.

This flow either signs in an existing user or auto-creates a free-tier account.

## When to use

- The user has no API key and no `LITEHOST_API_KEY` environment variable
- The agent needs to authenticate the user without opening a browser
- A previously issued OTP key has expired (7-day TTL)

DO NOT use this if the user already has a valid API key. Use the key directly.

---

## Step 1 — Request OTP code

```
POST /v1/auth/otp/request
Content-Type: application/json
```

No authentication required.

| Field | Type | Required | Description |
|---|---|---|---|
| email | string | yes | The user's email address. |

```bash
curl -X POST https://connect.litehost.io/v1/auth/otp/request \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com"}'
```

### Response (200)

```json
{
  "status": "success",
  "message": "Verification code sent. Check your email."
}
```

The response is identical whether or not the email has an existing account (prevents enumeration).

### Rate limits

- 3 requests per email per hour
- 10 requests per IP per hour

After requesting, tell the user: "A 6-digit code was sent to your email. Please enter it here."

---

## Step 2 — Verify code and get API key

```
POST /v1/auth/otp/verify
Content-Type: application/json
```

No authentication required.

| Field | Type | Required | Description |
|---|---|---|---|
| email | string | yes | Same email used in Step 1. |
| code | string | yes | The 6-digit code from the email. Must be exactly 6 digits. |

```bash
curl -X POST https://connect.litehost.io/v1/auth/otp/verify \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "code": "482931"}'
```

### Response (200)

```json
{
  "status": "success",
  "data": {
    "email": "you@example.com",
    "apiKey": "lh_live_a1b2c3d4e5f6...",
    "expiresAt": "2026-04-09T14:23:00.000Z",
    "message": "Account ready. This API key expires in 7 days. Use it as your Bearer token — it is shown only once."
  }
}
```

### After receiving the key

1. Store it as `LITEHOST_API_KEY` in the environment or a secrets manager.
2. Tell the user: "You're signed in. Your API key expires on {expiresAt}."
3. The key is shown only once — it is NOT visible in the Litehost dashboard.
4. If no account existed, one was auto-created on the free plan.

---

## Key Expiry

OTP-generated keys are valid for **7 days**. When the key expires:

1. Any request returns `401 Unauthorized`.
2. Re-run the full OTP flow (Step 1 + Step 2) to get a fresh key.
3. The account is NOT re-created — only a new key is issued.

---

## Error Handling

| Step | Status | Action |
|---|---|---|
| Request | 400 | Email missing or invalid format. Ask user to provide a valid email. |
| Request | 429 | Rate limited. Wait and retry later. Tell the user. |
| Verify | 400 | Email or code missing/malformed. Check inputs. |
| Verify | 401 | Code invalid, expired (10 min TTL), or already used. Request a new code via Step 1. |
| Verify | 500 | Server error. Safe to retry the full OTP flow. |
