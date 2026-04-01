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
| 401 | — | API key missing, invalid, or expired. | Follow `utils/auth.md`. |
| 403 | `PROJECT_LIMIT_REACHED` | Active project count equals the plan limit. | Follow `utils/quotas.md`. |
| 403 | `STORAGE_LIMIT_REACHED` | Total storage of active projects equals the plan limit. | Follow `utils/quotas.md`. |
| 404 | — | Resource not found. | Verify the ID with the user. If unknown, list the resource type (projects, domains, workspaces) and let them pick. |
| 409 | — | Slug conflict — another project already uses that slug. | Ask the user for a different slug and retry. |
| 422 | — | Request body validation failed. | Check field types and constraints against the action file, fix the request, and retry. |

## Rules

- NEVER silently swallow errors. Always inform the user what went wrong and why.
- NEVER retry the same request without changing something (different parameter, resolved auth, etc.).
- When a 403 limit error occurs, NEVER just tell the user to upgrade. Follow the full quota flow in `utils/quotas.md` first.
