# Error Handling

All error responses follow this shape:

```json
{
  "status": "error",
  "error": "Human-readable message",
  "code": "OPTIONAL_ERROR_CODE"
}
```

## Error Reference

| Status | Code | Meaning | Required Action |
|---|---|---|---|
| 400 | `ZIP_MULTIPLE_HTML` | ZIP contains multiple HTML files and no `zipIndexHtmlPath` was provided. | Read the `htmlPaths` array from the response. Present the paths to the user. Ask which is the homepage. Retry with `zipIndexHtmlPath` set to their choice. |
| 401 | — | API key missing, invalid, or expired. | Follow `utils/auth.md`. If the key was OTP-generated it may have expired (7-day TTL) — re-run the OTP flow. |
| 403 | `FREE_TIER_RESTRICTED` | Endpoint requires a paid plan (starter or higher). Response includes `requiredTier`. | Tell the user: "This feature requires a paid plan ({requiredTier} or higher). You can upgrade at https://litehost.io/dashboard." DO NOT retry. |
| 403 | `PROJECT_LIMIT_REACHED` | Active project count equals the plan limit. | Follow `utils/quotas.md`. |
| 403 | `STORAGE_LIMIT_REACHED` | Total storage of active projects equals the plan limit. | Follow `utils/quotas.md`. |
| 404 | — | Resource not found. | Verify the ID with the user. If unknown, list the resource type (projects, domains, workspaces) and let them pick. |
| 409 | — | Slug conflict — another project already uses that slug. | Ask the user for a different slug and retry. |
| 409 | `ALREADY_CLAIMED` | Temp project was already claimed. | The project is already in the user's account. No action needed — inform the user. |
| 422 | — | Request body validation failed. | Check field types and constraints against the action file, fix the request, and retry. |
| 429 | — | Rate limit exceeded. | Wait for the duration indicated in the `Retry-After` header (or a reasonable backoff) and retry. Tell the user about the wait. |

## Free-Tier Restricted Endpoints

These endpoints return `403 FREE_TIER_RESTRICTED` for free-plan users:
- PUT `/v1/projects/{id}` (push new version)
- PATCH `/v1/projects/{id}` (update settings)
- GET `/v1/projects/{id}/status` (deployment history)
- POST `/v1/projects/{id}/archive`
- POST `/v1/projects/{id}/unarchive`
- GET `/v1/domains`, GET `/v1/domains/{id}`
- GET/POST/PATCH/DELETE `/v1/workspaces`

When hitting this error, check `data.plan.tier` from `GET /v1/user`. If the user is on `free`, they must upgrade before using these endpoints.

## Rules

- NEVER silently swallow errors. Always inform the user what went wrong and why.
- NEVER retry the same request without changing something (different parameter, resolved auth, etc.).
- When a 403 limit error occurs, NEVER just tell the user to upgrade. Follow the full quota flow in `utils/quotas.md` first.
- When a 403 `FREE_TIER_RESTRICTED` error occurs, DO tell the user to upgrade — there is no workaround.
