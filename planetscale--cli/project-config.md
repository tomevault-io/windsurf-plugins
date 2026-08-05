---
trigger: always_on
description: > **Developing the CLI?** The API client is vendored at `internal/planetscale/`;
---

# PlanetScale CLI — agent guide

> **Developing the CLI?** The API client is vendored at `internal/planetscale/`;
> this repo **no longer depends on `planetscale-go`**. Read `doc/api-client.md`
> before touching API-facing code. The rest of this file is about *using* `pscale`.

For **any** automated agent or script using `pscale`. Always pass **`--format json`**. Substitute placeholders from the user's request or from prior command output (`org list`, `database list`, `branch list`).

If you only have the installed `pscale` binary, start here:

```bash
pscale agent-guide --format json
pscale auth check --format json
```

Use direct CLI automation for shell commands and scripts. Use the hosted PlanetScale MCP server for MCP clients.

This file documents **how to invoke `pscale`**. For database assessment, safety review, and operational workflows, install the [PlanetScale skills pack](https://github.com/planetscale/skills) (`14-pscale-cli-automation` covers CLI automation; `00-safe-orchestrator` runs the full review). In application repositories, add a separate **project** `AGENTS.md` with org, database, branch, and approval rules (see skill `09-mcp-agent-operating-model` in that repo).

| Placeholder | Meaning |
|-------------|---------|
| `<org>` | Organization name |
| `<database>` | Database name |
| `<branch>` | Branch name (pick one with `"ready": true` from `branch list`) |

## Flag placement

- **`--org`** is a flag on resource subcommands (`database`, `branch`, `sql`, `api`, …). It is **not** on root `pscale` — `pscale --org …` fails.
- **`--format json`** is a global flag. It can go on `pscale` or on the subcommand.
- Commands with positional args (`sql`, `branch list`, …): put **positionals first**, then flags.

```bash
# Correct
pscale auth check --format json
pscale org list --format json
pscale database list --org <org> --format json
pscale branch list <database> --org <org> --format json
pscale sql <database> <branch> --org <org> --format json --query "SELECT 1"

# Also valid — global --format
pscale --format json database list --org <org>

# Wrong — unknown flag: --org
pscale --org <org> database list --format json
```

## Workflow

1. **Guide** — discover machine-readable conventions when you do not have this file:

   ```bash
   pscale agent-guide --format json
   ```

2. **Auth** — check before anything else:

   ```bash
   pscale auth check --format json
   ```

   `"status": "ok"` and `"authenticated": true` with no blocking `issues` means proceed. `"status": "action_required"` exits non-zero — log in, pick an org, or fix credentials (see `issues` and `next_steps`).

3. **Login** (when not authenticated):

   ```bash
   pscale auth login --format json
   ```

   Pending JSON is written to **stderr** while waiting; **stdout** has a single final JSON object when login completes (`status: ok` or `action_required` if org setup fails after credentials are saved). Fields include `verification_url`, `user_code`, and `browser_opened`. Open `verification_url` manually if the browser does not open. Do not retry login in a loop without browser access.

4. **Organization** — use `"organization"` from `auth check`, ask the user, or list orgs:

   ```bash
   pscale org list --format json
   ```

   Pass `--org <org>` on resource commands (`database`, `branch`, `sql`, `api`, …). Not on `org list`.

5. **Discover resources** before SQL:

   ```bash
   pscale database list --org <org> --format json
   pscale branch list <database> --org <org> --format json
   ```

6. **Query** (read-only default):

   ```bash
   pscale sql <database> <branch> --org <org> --format json --query "SELECT 1"
   ```

## Flags

| Flag | Purpose |
|------|---------|
| `--format json` | JSON on stdout |
| `--org <org>` | Organization (on resource subcommands only) |
| `--api-url` | Non-production API base URL — pass on every command when not using production |

## JSON errors

With `--format json`, any command that fails prints exactly one JSON envelope on **stdout**:

```json
{
  "status": "error",
  "error": "human-readable message",
  "issues": [{ "code": "NOT_FOUND", "message": "human-readable message" }],
  "next_steps": ["pscale org list --format json", "pscale database list --org <org> --format json"]
}
```

- `status` is `"error"` or `"action_required"`. `action_required` means an agent can recover by following `next_steps` (log in, ask the user for approval, fix the invocation). Exit code is `1` for `action_required` and `2` for `error`.
- `issues[].code` is stable and machine-readable; branch on it, not on message text.
- `next_steps` are concrete commands or instructions, ordered by likelihood.

Some commands add fields to this envelope (for example `query_kind` on destructive SQL or `migration_id` on imports) but `status`, `issues`, and `next_steps` are always present on failure.

| Code | Meaning |
|------|---------|
| `NO_AUTH` | Not authenticated or token expired; run `pscale auth login --format json` |
| `AUTH_INVALID` | Stored credentials rejected by the API; log in again |
| `SERVICE_TOKEN_INVALID` | Service token id/secret rejected; verify the values |
| `NO_ORG` | Authenticated but no organization configured |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [planetscale/cli](https://github.com/planetscale/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
