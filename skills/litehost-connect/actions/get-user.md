# Get User

Retrieve the authenticated user's email, plan tier, limits, and current quota usage.

## Endpoint

```
GET /v1/user
```

## Example

```bash
curl https://connect.litehost.io/v1/user \
  -H "Authorization: Bearer $LITEHOST_API_KEY"
```

## Success Response (200)

```json
{
  "status": "success",
  "data": {
    "email": "user@example.com",
    "plan": {
      "tier": "creator",
      "limits": {
        "maxProjects": 10,
        "maxStorageBytes": 2147483648,
        "maxFilesPerProject": 200,
        "maxVisitorsPerMonth": 100000
      }
    },
    "quota": {
      "projects": { "used": 3, "limit": 10, "unlimited": false },
      "storage": { "usedBytes": 52428800, "limitBytes": 2147483648, "unlimited": false }
    }
  }
}
```

## Key Fields

| Field | Meaning |
|---|---|
| `plan.tier` | One of: `free`, `starter`, `creator`, `pro`, `scale`. |
| `quota.projects.used` | Active (non-archived) project count. |
| `quota.projects.limit` | Max allowed. `-1` means unlimited. |
| `quota.storage.usedBytes` | Bytes used by active projects only. |
| `quota.storage.limitBytes` | Max allowed. `-1` means unlimited. |
| `unlimited` | When `true`, no cap on that resource. |

## When to call

ALWAYS call this endpoint before any operation that creates or restores a project:
- Before `create-project.md`
- Before `unarchive-project.md`

Also call when the user asks about their plan, usage, or remaining capacity.

## Error Handling

| Status | Action |
|---|---|
| 401 | Follow `utils/auth.md`. |
