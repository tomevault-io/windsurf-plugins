---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo is the **AgEnFK framework itself** — a TypeScript monorepo that ships:
- An MCP server + REST/WebSocket API (`packages/server`)
- A CLI (`packages/cli`, exposed at the repo root via `bin/agenfk.js`)
- A React Kanban UI (`packages/ui`)
- A `core` package with shared types/lifecycle logic, a `storage-sqlite` plugin, and `telemetry`
- A `create` package (project scaffolder) and per-client rule bundles in `clauderules/`, `cursorrules/`, `codexrules/`, `geminirules/`, plus `commands/` and `skills/` for slash-command/skill packs
- Installer/uninstaller scripts (`scripts/install.mjs`, `scripts/uninstall.mjs`) that copy these rule bundles, MCP config, and PreToolUse hooks into each AI client's config dir

Note: the repo's own CLAUDE.md (this file) is *not* the file that gets installed onto end-user machines — `clauderules/CLAUDE.md` is. The installer merges that bundle plus equivalents (`AGENTS.md`, `.cursor/rules/*.mdc`, etc.) into the user's client config. Keep `clauderules/CLAUDE.md` in sync with `SKILL.md`/`SDLC.md` when changing workflow rules.

The release commands (`/agenfk-release`, `/agenfk-release-beta`, `/agenfk-release-hub`) are **repo-private**: they cut releases of the framework itself, so they live in `.claude/commands/` and are NOT shipped to users (guarded by `packages/cli/src/test/release-commands-private.test.ts`; the installer deletes stale global copies on upgrade). They are exempt from the active-task requirement — do not create or require an AgEnFK task when executing them.

## Common commands

Build everything (core/storage/telemetry first, then cli/server/ui):
```
npm run build
```

Build a single package:
```
npm run build -w packages/server
```

Run all tests (vitest, node env, file-parallelism off — tests share filesystem state):
```
npm test
```

Run a single test file or filter by name:
```
npx vitest run packages/server/src/test/foo.test.ts
npx vitest run -t "name pattern"
```

Coverage (gated at 80% for `core`, `storage-sqlite`, `server` per `vitest.config.ts`):
```
npm run test:coverage
```

UI is excluded from the root vitest run. For UI tests + coverage:
```
npm run test:ui:coverage
```

UI dev server / lint:
```
cd packages/ui && npm run dev
cd packages/ui && npm run lint
```

Installer / services (used by end users, but useful when iterating on install logic):
```
npm run install:framework      # copies rules + MCP config into ~/.claude, ~/.cursor, etc.
npm run uninstall:framework
npm run start:services         # launches the API server + UI
```

The CLI binary `bin/agenfk.js` is a thin shim that resolves to `packages/cli`. Build CLI before invoking it directly: `npm run build -w packages/cli`.

Bump the monorepo version everywhere at once (root + all `packages/*/package.json`, including internal `@agenfk/*` dep references across dependencies/devDependencies/peerDependencies):
```
node scripts/bump-version.mjs <new-version>   # e.g. 1.1.8-beta.6
npm install --package-lock-only               # regenerate the lockfile
git add . && git commit -m "chore: bump version to <new-version>"
```
The old version is read from the root `package.json`; commit the manifest changes and the regenerated lockfile together so they never drift. The release commands (`/agenfk-release`, `/agenfk-release-beta`) use this flow.

## Architecture (big picture)

**Single Owner**: the API server in `packages/server` is the only writer of state. CLI, MCP clients, and UI all go through its REST endpoints; updates fan out over Socket.io. Never read `.agenfk/db.sqlite` directly from other packages — go through the storage interface in `core`.

**Storage**: SQLite-only via `better-sqlite3` (`packages/storage-sqlite`). The repo previously supported `db.json`; existing JSON DBs are auto-migrated by the installer. WAL mode + indexed schema.

**Workflow engine** (`packages/core` + enforced by `packages/server`):
- Items have type (EPIC / STORY / TASK / BUG) and move through a configurable **Flow** of `FlowStep`s (default: TODO → IN_PROGRESS → REVIEW → TEST → DONE; per-project flows override this).
- Forward transitions are gated by `validate_progress(itemId, evidence, command?)`. The final step's `command` defaults to `project.verifyCommand`.
- DONE is unreachable by direct `update_item({ status: "DONE" })` — the server blocks it; only `validate_progress` on the final step can land it.
- `workflow_gatekeeper(intent, itemId?)` is the pre-edit authorization check; it surfaces the active step's `exitCriteria` so the caller knows what to satisfy before `validate_progress`.

**MCP surface**: the server registers MCP tools (`mcp__agenfk__*`) backed by the same handlers as the REST endpoints. Tool definitions live in `packages/server/src` alongside the Express routes. Adding a new MCP tool means: handler in core, REST route in server, MCP tool registration in server, and (usually) a CLI subcommand in `packages/cli` for the fallback path.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cglab-public/agenfk](https://github.com/cglab-public/agenfk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
