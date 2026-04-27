---
trigger: always_on
description: When making changes that affect published packages, create a changeset file:
---

# Project Guidelines

## Finalizing Changes

When making changes that affect published packages, create a changeset file:

```bash
cat > .changeset/<short-description>.md << 'EOF'
---
"@runfusion/fusion": patch
---

Short description of the change.
EOF
```

Bump types:
- **patch**: bug fixes, internal changes
- **minor**: new features, new CLI commands, new tools
- **major**: breaking changes

Include the changeset file in the same commit as the code change. The filename should be a short kebab-case description (e.g., `fix-merge-conflict.md`, `add-retry-button.md`).

Only create changesets for changes that affect the published `@runfusion/fusion` package — user-facing features, bug fixes, CLI changes, tool changes. Do NOT create changesets for internal docs (AGENTS.md, README), CI config, or refactors that don't change behavior.

## Package Structure

- `@fusion/core` — domain model, task store (private, not published)
- `@fusion/dashboard` — web UI + API server (private, not published)
- `@fusion/engine` — AI agents: triage, executor, reviewer, merger, scheduler (private, not published)
- `@runfusion/fusion` — CLI + pi extension (published to npm)

Only `@runfusion/fusion` is published. The others are internal workspace packages.

## Storage Model

Fusion uses a hybrid storage architecture: structured metadata lives in SQLite (`.fusion/fusion.db`) while large blob files (PROMPT.md, attachments) remain on the filesystem under `.fusion/tasks/{ID}/`. The database runs in WAL mode for concurrent access.

See [docs/storage.md](./docs/storage.md) for the full storage architecture documentation.

## Multi-Project Support

Fusion supports multiple projects with a central registry at `~/.fusion/fusion-central.db`. Each project has its own SQLite database at `.fusion/fusion.db`. See [docs/multi-project.md](./docs/multi-project.md) for details on:
- CentralCore API and project registration
- Isolation modes (in-process, child-process)
- Global concurrency management

## Testing

```bash
VITEST_MAX_WORKERS=4 pnpm test  # run all tests
pnpm build         # build all packages
```

Tests are required. Typechecks and manual verification are not substitutes for real tests with assertions.

## Port 4040 is Reserved

Port 4040 is the production dashboard port. A user's live dashboard session is typically running there. **Agents must NEVER:**
- Run `kill`, `kill -9`, `pkill`, or `killall` against processes on port 4040
- Start a test server on port 4040 — always use `--port 0` for random free port

## Engine Process Rules

The engine (`packages/engine`) runs the executor, merger, scheduler, IPC host, and dashboard-facing activity loop on a single Node event loop. **Blocking that loop stalls every task concurrently in-flight.**

### Never use `execSync` for User-Configured Commands

`execSync` blocks the entire event loop until the child process exits. Any command from project settings — `testCommand`, `buildCommand`, `workflow step scripts`, etc. — **must** run via `promisify(exec)` with `timeout`. Never use `execSync` for user-configured commands.

```ts
import { exec } from "node:child_process";
import { promisify } from "node:util";
const execAsync = promisify(exec);

const { stdout, stderr } = await execAsync(command, {
  cwd: worktreePath,
  timeout: 120_000,
  maxBuffer: 10 * 1024 * 1024,
});
```

`execSync` is only acceptable for short, deterministic git plumbing (`git rev-parse`, `git branch -d`, `git worktree remove`, etc.). When in doubt, use async.

## Git Conventions

- Commit messages: `feat(FN-XXX):`, `fix(FN-XXX):`, `test(FN-XXX):`
- One commit per step (not per file change)
- Always include the task ID prefix

## Node Dashboard

Fusion has a Node Dashboard view for managing mesh network nodes. See [docs/architecture.md](./docs/architecture.md) for dashboard components and API endpoints.

**Node Settings Sync API Endpoints:**

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/nodes/:id/settings` | Fetch settings from a remote node |
| POST | `/api/nodes/:id/settings/push` | Push local settings to a remote node |
| POST | `/api/nodes/:id/settings/pull` | Pull settings from a remote node |
| GET | `/api/nodes/:id/settings/sync-status` | Get sync status and diff summary |
| POST | `/api/nodes/:id/auth/sync` | Sync model auth credentials |
| POST | `/api/settings/sync-receive` | Receive pushed settings (inbound) |
| POST | `/api/settings/auth-receive` | Receive auth credentials (inbound) |
| GET | `/api/settings/auth-export` | Export local auth credentials |

All remote node endpoints require the target node to have an `apiKey` configured. Inbound endpoints validate the `Authorization: Bearer <apiKey>` header against the local node's apiKey.

## Pi Extension (`packages/cli/src/extension.ts`)

The pi extension provides tools and a `/fn` command for interacting with fn from within a pi session. It ships as part of `@runfusion/fusion`.

**Update the extension when:**
- CLI commands change (behavior, flags, or output)
- Task store / Agent store API changes (method signatures or behavior)
- New user-facing features are added that chat agents should be able to use


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Runfusion/Fusion](https://github.com/Runfusion/Fusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
