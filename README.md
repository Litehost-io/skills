# Litehost Connect — Agent Skill

Agent skill for deploying and managing projects on [litehost.io](https://litehost.io) through the Connect API. Works with Claude Code, Cursor, and any agent framework that supports file-based skill loading.

## What's inside

```
Skill.md                ← Entry point: routing table, auth gate, quota gate
actions/
  otp-sign-in.md        ← POST /v1/auth/otp/request + /verify  (keyless auth)
  temp-project.md       ← POST /v1/projects/temp + /claim       (no auth needed)
  create-project.md     ← POST /v1/projects
  replace-project.md    ← PUT  /v1/projects/{id}                (push new files)
  update-project.md     ← PATCH /v1/projects/{id}               (change settings)
  list-projects.md      ← GET  /v1/projects
  get-project.md        ← GET  /v1/projects/{id}
  deployment-history.md ← GET  /v1/projects/{id}/status
  delete-project.md     ← DELETE /v1/projects/{id}
  archive-project.md    ← POST /v1/projects/{id}/archive
  unarchive-project.md  ← POST /v1/projects/{id}/unarchive
  domains.md            ← GET  /v1/domains, /v1/domains/{id}, assign/remove
  workspaces.md         ← CRUD /v1/workspaces
  get-user.md           ← GET  /v1/user (plan, quota)
utils/
  auth.md               ← API key resolution (dashboard keys + OTP)
  errors.md             ← Error code reference + required agent actions
  quotas.md             ← Limit-reached handling sequence
```

## Prerequisites

You need an API key. Two ways to get one:

- **Dashboard** — generate a permanent key at [litehost.io/dashboard → Integrations](https://litehost.io/dashboard).
- **OTP sign-in** — the skill can authenticate you via email code, no dashboard needed. See `actions/otp-sign-in.md`.

If you already have a key, expose it as an environment variable:

```bash
export LITEHOST_API_KEY="lh_live_..."
```

No key at all? The skill can still do instant uploads via `POST /v1/projects/temp` (no auth required, 15-min expiry).

---

## Quick Install

```bash
npx skills add litehost-io/skills
```

This downloads the skill and registers it with your agent. Works with Claude Code, Cursor, and any compatible agent framework.

---

## Manual Installation

If you prefer to install manually or need to customize the setup:

### Claude Code

**1. Download the skill files**

```bash
curl -L https://github.com/Litehost-io/skills/archive/refs/heads/main.tar.gz | tar -xz
mv skills-main ~/.claude/skills/litehost-connect
```

**2. Add to your project's `CLAUDE.md`**

```markdown
## Skills

When the user asks to deploy, host, publish, or manage a project on Litehost,
read and follow the skill at `~/.claude/skills/litehost-connect/SKILL.md`.
```

### Cursor

**1. Download the skill files**

```bash
curl -L https://github.com/Litehost-io/skills/archive/refs/heads/main.tar.gz | tar -xz
mv skills-main .cursor/skills/litehost-connect
```

**2. Add a rule file at `.cursor/rules/litehost.mdc`**

```markdown
---
description: Deploy and manage projects on litehost.io
globs:
alwaysApply: false
---

When the user asks to deploy, host, publish, update, or manage a project on Litehost,
read and follow the skill at `.cursor/skills/litehost-connect/SKILL.md`.
```

### General Agents

1. Download and extract the skill files into a path the agent can read.
2. Load `SKILL.md` as the entry point when the user's intent matches deployment or hosting.
3. The agent reads sub-files (`actions/*.md`, `utils/*.md`) on demand — each is self-contained with endpoint, parameters, curl examples, response shapes, and error handling.

Add this to your agent's system prompt or tool description:

```
When the user asks to deploy, host, or manage a project on Litehost, read
<path-to>/SKILL.md and follow its instructions. It references sub-files in
actions/ and utils/ — read each one as needed before making API calls.
All requests go to https://connect.litehost.io with Bearer token auth.
```

---

## API Reference

Base URL: `https://connect.litehost.io`
