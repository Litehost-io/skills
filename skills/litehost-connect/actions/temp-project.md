# Temporary Project

Upload files and get a live URL instantly — no API key required. The project expires in 15 minutes unless claimed.

## When to use

- The user wants the fastest path to a live URL with zero setup
- The user is not yet authenticated and wants to try before signing in
- Quick previews or one-off file sharing

For permanent projects, use `create-project.md` instead (requires auth).

---

## Create Temporary Project

```
POST /v1/projects/temp
Content-Type: multipart/form-data
```

No authentication required.

### Limits

- Max 5 files per request
- Total upload size: 2 MB
- Rate limit: 3 uploads/minute, 5 uploads/hour per IP

### Parameters

| Field | Type | Required | Description |
|---|---|---|---|
| files | file[] | yes | One or more files (max 5, total 2 MB). |
| zipIndexHtmlPath | string | no | Homepage path inside a ZIP when it contains multiple HTML files. |
| asFileBundle | boolean | no | Set `true` to force file-bundle mode. |

### Example

```bash
curl -X POST https://connect.litehost.io/v1/projects/temp \
  -F "files=@index.html"
```

### Response (200)

```json
{
  "status": "success",
  "data": {
    "projectId": "uuid",
    "slug": "blue-fox-42",
    "url": "https://blue-fox-42.litepage.site",
    "claimToken": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "expiresAt": "2026-04-03T01:30:00.000Z"
  }
}
```

After a successful upload:
1. Return the `url` to the user immediately.
2. Tell them: "This link expires in 15 minutes. Want to keep it permanently?"
3. If yes, proceed to claim (below). This requires authentication — if the user has no API key, run the OTP flow in `otp-sign-in.md` first.

---

## Claim Temporary Project

```
POST /v1/projects/claim/{claimToken}
```

Requires authentication (Bearer token).

Permanently transfers the temp project to the user's account.

- **Free plan** — expiry extended to 7 days from claim time.
- **Paid plans** — expiry removed; project becomes permanent.

### Pre-flight

Check quota via `GET /v1/user` before claiming. The API rejects with 403 if project or storage limits would be exceeded.

### Example

```bash
curl -X POST https://connect.litehost.io/v1/projects/claim/{claimToken} \
  -H "Authorization: Bearer $LITEHOST_API_KEY"
```

### Response (200)

```json
{
  "status": "success",
  "data": {
    "projectId": "uuid",
    "slug": "blue-fox-42",
    "url": "https://blue-fox-42.litepage.site",
    "expiresAt": "2026-04-10T01:15:00.000Z"
  }
}
```

`expiresAt` is `null` for paid plans (permanent).

### Error Handling

| Status | Code | Action |
|---|---|---|
| 400 | `ZIP_MULTIPLE_HTML` | Present `htmlPaths` to user, retry with `zipIndexHtmlPath`. |
| 401 | — | Follow `utils/auth.md`. |
| 403 | `PROJECT_LIMIT_REACHED` | Follow `utils/quotas.md`. |
| 403 | `STORAGE_LIMIT_REACHED` | Follow `utils/quotas.md`. |
| 404 | — | Claim token not found or expired. The temp project is gone. |
| 409 | `ALREADY_CLAIMED` | Project was already claimed. No action needed. |
| 429 | — | Rate limited. Wait and retry. |
