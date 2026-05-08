---
trigger: always_on
description: Termlings agents are autonomous Claude Code or Codex sessions that collaborate through a shared local workspace.
---

# Termlings Agents

Termlings agents are autonomous Claude Code or Codex sessions that collaborate through a shared local workspace.

## Current model

- No sim engine.
- No map/walk/place/chat actions.
- No Pi adapter.
- Claude Code and Codex are supported agent runtimes.
- File-backed JSON state in `.termlings/`.

## Implementation policy (current phase)

- Do not add backward-compatibility layers unless explicitly requested.
- Do not add fallback paths/aliases for old file names, commands, or data formats.
- Prefer one canonical path/format and update templates/docs/code together.

## App implementation contract

Termlings apps are the main capability unit. New apps should follow one consistent structure.

### App registration

Every new app should be wired through:

- `src/apps/core-apps.json` for app metadata, help ordering, TUI/slash contribution metadata, and agent visibility
- `src/commands/index.ts` for routing
- `src/engine/apps.ts` and system-context/help surfaces when the app affects visibility or access

### Code layout

Apps do not currently live in a single `src/apps/<app>` folder. The canonical split is:

```text
src/commands/<app>.ts         # CLI contract + routing
src/engine/<app>.ts           # file-backed state, provider logic, mutations, reads
src/engine/__tests__/<app>.test.ts
docs/<APP>.md                 # operator/agent docs
```

If an app needs shared helpers, add a focused helper file rather than inventing a second app-local folder layout.

### Storage layout

Every app should own one canonical store namespace under:

```text
.termlings/store/<app>/
```

Rules:

- keep the source of truth file-backed and local-first
- prefer inspectable JSON / JSONL
- use append-only history where auditability matters
- do not create duplicate state in multiple app namespaces
- if scheduling is needed, integrate with the shared scheduler instead of inventing an app-local timing engine
- if activity is meaningful, emit through the shared activity system instead of app-local TUI-only state

### App context and visibility

- App access is globally gated in `.termlings/workspace.json.apps.defaults`
- Per-agent access is narrowed in `.termlings/agents/<slug>/SOUL.md` via frontmatter `apps:`
- If an agent does not have access to an app, that app should not appear in:
  - system context
  - top-level agent help
  - TUI tabs owned by that app
  - app-backed `brief` sections

## App CLI contract

Newer app commands are JSON-first. Do not design new app CLIs around positional human flags.

Canonical pattern:

```bash
termlings <app> schema [<action>] [--json]
termlings <app> <action> --params '{"..."}' --json
printf '%s\n' '{...}' | termlings <app> <action> --stdin-json [--params '{"..."}'] [--json]
```

### Required conventions

- `schema`
  - first-class contract inspection for the whole app or one action
- `--params`
  - JSON object for ids, filters, windows, query selectors, pagination, and read-side targeting
- `--stdin-json`
  - canonical input for create/update/schedule/publish/write actions
- `--json`
  - machine-readable output for reads and writes

Use `src/commands/app-api.ts` for new JSON-first app commands unless there is a strong reason not to.

### `--dry-run`

For new write/publish/send/destructive app actions, support `--dry-run` when it is meaningful.

Examples:

- publish
- send
- schedule create
- archive/delete
- sync actions that may mutate local state or trigger external side effects

If `--dry-run` does not make sense for an action, that should be a deliberate choice, not an omission by default.

### Help and docs rules

- Agent-facing help should teach the canonical JSON contract, not alternate human-friendly flag styles
- Avoid adding extra human sugar if it clutters `--help`, system context, or app docs
- Show `schema`, `--params`, and `--stdin-json` examples in docs
- Document app env requirements in the app doc and prefer `.termlings/.env` for Termlings-owned provider config

### Action design rules

- read/list/show/report actions:
  - prefer `--params` + `--json`
- create/update/note/schedule/publish actions:
  - prefer `--stdin-json`
- avoid ad hoc positional-argument variants for app actions
- avoid mixing multiple CLI styles inside the same app
- do not hide required structure in prose; expose it through `schema`

## Architecture

```text
termlings (CLI)
  ├─ termlings brief          # complete workspace snapshot
  ├─ termlings claude         # launch agent session
  ├─ termlings org-chart      # discover org structure + active status
  ├─ termlings message ...    # direct messaging
  ├─ termlings conversation   # read conversation history
  ├─ termlings task ...       # task management
  ├─ termlings email ...      # email workflow wrapper (Himalaya)
  ├─ termlings calendar ...   # calendar viewing
  ├─ termlings --server       # secure local HTTP API server
  └─ termlings                # terminal workspace UI (default)

Workspace state: <project>/.termlings/
  .env
  emails.json
  sessions/*.json
  store/messages/*
  store/tasks/tasks.json
  store/calendar/calendar.json
  agents/<agent-id>/SOUL.md
```

## Default Team Structure

The template includes a 5-person bootstrapped team structure:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomtev/termlings](https://github.com/tomtev/termlings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
