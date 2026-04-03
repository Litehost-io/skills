---
name: litehost-connect
description: >
  Deploy, manage, and update projects on litehost.io via the Connect API.
  Activate when the user asks to deploy, host, publish, upload, update, list,
  delete, archive, or manage any web project, site, file, or domain on Litehost.
  Also activate for sign-in, quota checks, workspace management, and custom domain assignment.
user-invocable: true
argument-hint: "[file-or-folder]"
allowed-tools: Bash
---

# Litehost Connect

Skill version: 3.0.0

Base URL: `https://connect.litehost.io`

---

## Authentication

Read `utils/auth.md` and follow it before making any authenticated API call.

Two paths to get an API key:
1. **Dashboard key** — user generates a permanent key at https://litehost.io/dashboard → Integrations.
2. **OTP sign-in** — run `actions/otp-sign-in.md` to get a 7-day key via email code. No browser needed.

If the user has no key and no `LITEHOST_API_KEY` env var, use the OTP flow automatically.

---

## Quota Gate

Before any operation that **creates** or **restores** a project (`create-project`, `unarchive-project`, `claim`), call `GET /v1/user` and check quotas. If limits are reached, follow the full flow in `utils/quotas.md`. NEVER skip this step.

---

## Free-Tier Awareness

Many endpoints require a paid plan (starter or higher). If the user is on the free tier and hits a `FREE_TIER_RESTRICTED` error, tell them the feature requires upgrading and link to https://litehost.io/dashboard. See `utils/errors.md` for the full list of restricted endpoints.

Free-tier users CAN:
- Create projects (`POST /v1/projects`)
- Upload temp projects (`POST /v1/projects/temp`) — no auth needed
- Claim temp projects (`POST /v1/projects/claim/{token}`)
- List projects, get project details, delete projects
- Use OTP sign-in
- Check user/plan/quota info

Free-tier users CANNOT:
- Push new versions, update settings, view deployment history
- Archive or unarchive projects
- Use domains or workspaces

---

## Action Routing

Map the user's intent to exactly one action file. Read that file and execute its instructions.

| User says | Action file |
|---|---|
| "sign in", "log in", "I don't have an API key", "authenticate" | `actions/otp-sign-in.md` |
| "quick upload", "just host this temporarily", "preview this" | `actions/temp-project.md` |
| "deploy this", "host this", "publish this", "upload this" | `actions/create-project.md` |
| "update my site", "push new version", "redeploy", "replace content" | `actions/replace-project.md` |
| "rename project", "change slug", "make private", "set password", "update settings", "set SEO", "change title", "set expiry" | `actions/update-project.md` |
| "list my projects", "show projects", "what do I have hosted" | `actions/list-projects.md` |
| "get project details", "show project info", "project status" | `actions/get-project.md` |
| "show deploy history", "list versions", "deployment log" | `actions/deployment-history.md` |
| "delete project", "remove project" | `actions/delete-project.md` |
| "archive project", "free up space", "deactivate project" | `actions/archive-project.md` |
| "unarchive project", "restore project", "reactivate project" | `actions/unarchive-project.md` |
| "list domains", "show domains", "assign domain", "remove domain", "custom domain" | `actions/domains.md` |
| "list workspaces", "create workspace", "rename workspace", "delete workspace", "organize projects" | `actions/workspaces.md` |
| "check my plan", "how much space do I have", "check quota", "usage" | `actions/get-user.md` |

### Deploy without auth — fast path

If the user wants to deploy but has no API key and wants the fastest path:
1. Use `actions/temp-project.md` to upload instantly (no auth).
2. Offer to claim the project permanently.
3. If they want to claim, run `actions/otp-sign-in.md` first to get a key, then claim.

---

## When NOT to use this skill

DO NOT activate for:
- General web development questions unrelated to hosting or deployment
- Requests about other hosting providers (Vercel, Netlify, AWS, etc.)
- Local development server setup
- DNS configuration outside Litehost custom domains
- Billing or payment questions — direct the user to https://litehost.io/dashboard

---

## File Detection Rules

When the user provides files to deploy, determine the upload strategy:

1. Single HTML file or folder with HTML → static site
2. ZIP with exactly one HTML file → static site (that HTML is the homepage)
3. ZIP with multiple HTML files → MUST set `zipIndexHtmlPath`. Try common patterns first (`index.html`, `dist/index.html`). If ambiguous, ask the user.
4. ZIP with no HTML files → file bundle
5. Non-HTML files (PDF, images, code, docs) → file bundle
6. To force file-bundle mode regardless of content → set `asFileBundle=true`

---

## Output Rules

After every successful create, temp-upload, or push-version operation, ALWAYS return:
1. The live project URL
2. The project ID (for future updates)

---

## Error Handling

Read `utils/errors.md` for the complete error reference and required agent actions per error code.
