# Authentication

Every request to `https://connect.litehost.io` requires a Bearer token.

## Required Header

```
Authorization: Bearer lh_live_xxx
```

## Resolve the API Key

Follow these steps in order:

1. Read the `LITEHOST_API_KEY` environment variable.
2. If the variable is not set, ask the user: "Please provide your Litehost API key. You can generate one at https://litehost.io/dashboard → Integrations."
3. NEVER hardcode an API key in source files. Always use environment variables or a secrets manager.

## On 401 Unauthorized

If any request returns HTTP 401:

1. Tell the user: "Your API key is invalid or expired."
2. Direct them to verify or regenerate it at https://litehost.io/dashboard → Integrations.
3. DO NOT retry with the same key.
