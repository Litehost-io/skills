# Authentication

Every authenticated API request to `https://connect.litehost.io` requires a Bearer token.

```
Authorization: Bearer lh_live_xxx
```

## Resolve the API Key

Follow these steps in order:

1. Check the `LITEHOST_API_KEY` environment variable.
2. If set, use it. Proceed to make API calls.
3. If NOT set, run the OTP sign-in flow in `actions/otp-sign-in.md` to obtain a key without needing the dashboard.

NEVER hardcode an API key in source files. Always use environment variables or a secrets manager.

## Two Types of Keys

| Source | Lifetime | Visible in dashboard |
|---|---|---|
| Dashboard (Integrations page) | Never expires | Yes |
| OTP flow (`/v1/auth/otp/verify`) | 7 days | No |

## On 401 Unauthorized

If any request returns HTTP 401:

1. Check the error message. If it mentions expiry, the OTP key has expired.
2. If expired: re-run the OTP flow in `actions/otp-sign-in.md` to get a fresh key.
3. If not expired: the key is invalid or revoked. Ask the user to provide a new one or run the OTP flow.
4. DO NOT retry with the same key after a 401.

## No-Auth Endpoints

These endpoints do NOT require a Bearer token:
- `GET /health`
- `GET /v1`
- `POST /v1/auth/otp/request`
- `POST /v1/auth/otp/verify`
- `POST /v1/projects/temp`
