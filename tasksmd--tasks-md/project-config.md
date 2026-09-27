---
trigger: always_on
description: `tasks.md` defines the TASKS.md task-queue specification and ships the
---

# AGENTS.md - tasks.md Codebase Guide

## What This Repo Is

`tasks.md` defines the TASKS.md task-queue specification and ships the
supporting tools that make the format useful for humans and agents:

- `spec.md` is the canonical format specification.
- `README.md` is the landing page and quick start.
- `examples/` contains valid TASKS.md examples used as documentation fixtures.
- `packages/parser` parses task files, metadata, blockers, claims, and policies.
- `packages/lint` validates task files against the spec.
- `packages/mcp` exposes TASKS.md operations through the Model Context Protocol.
- `packages/cli` provides the `tasks` command-line interface.
- `commands/` contains the shared `/next-task` and `/lint-tasks` command variants
  for Claude Code, Codex, Cursor, Devin, Gemini CLI, and Windsurf.

## Repo Layout

```text
tasks.md/
+-- spec.md                         # Canonical TASKS.md format spec
+-- README.md                       # User-facing docs and quick start
+-- Agentfile.yaml                  # Repo-local agentbrew MCP manifest
+-- TASKS.md                        # Local task queue for this repo
+-- examples/                       # Valid TASKS.md example files
+-- commands/
|   +-- next-task.md                # Shared canonical /next-task source
|   +-- lint-tasks.md               # Shared canonical /lint-tasks source
|   +-- claude/skills/*/SKILL.md    # Claude Code skill variants
|   +-- codex/skills/*/SKILL.md     # OpenAI Codex skill variants
|   +-- cursor/*.md                 # Cursor command variants
|   +-- devin/skills/*/SKILL.md     # Devin skill variants
|   +-- gemini/*.toml               # Gemini CLI command variants
|   +-- windsurf/*.md               # Windsurf workflow variants
+-- packages/
|   +-- parser/                     # @tasks-md/parser TypeScript package
|   +-- lint/                       # @tasks-md/lint and tasks-lint binary
|   +-- mcp/                        # tasks-mcp server
|   +-- cli/                        # @tasks-md/cli and tasks binary
```

## Development

This is an npm workspace repo. Install dependencies once, then run commands from
the repo root unless a package README says otherwise.

```bash
npm install                         # Install workspace dependencies
npm run build                       # Type-check and build parser, lint, MCP, CLI
npm run build:site                  # Rebuild the static docs site
npm run lint                        # Run local TASKS.md lint via built package
npm test                            # Run all workspace tests
npm run test:cached                 # Cached test wrapper for repeated local runs
npx -y @tasks-md/lint TASKS.md      # Validate the public linter package path
```

Package-level commands also work with npm workspaces:

```bash
npm run build -w packages/parser
npm test -w packages/mcp
```

## Verification Gate

Before committing a normal change, run the smallest gate that covers the files
you touched, then run the full gate for cross-package or command/spec changes:

- **Docs-only / TASKS.md edits**: `npm run lint` and `npx -y @tasks-md/lint TASKS.md`.
- **Spec, parser, lint, MCP, CLI, or command behavior**: `npm run build`, `npm test`,
  `npm run lint`, and `npx -y @tasks-md/lint TASKS.md`.
- **README or website changes**: include `npm run build:site` when rendered site output
  could change.

Do not claim work is complete until the verification commands you ran have
finished successfully. Never bypass hooks with `--no-verify` unless the user has
explicitly approved that exact action in the current session.

## Release And CI Gotchas

The tag-triggered release (`.github/workflows/publish.yml`) and CI have sharp
edges that cost real debugging time — captured here so they don't recur:

- **npm OIDC Trusted Publishing needs npm >= 11.5.1.** Node 22 ships npm 10.x,
  which signs the provenance statement but **cannot authenticate the publish via
  OIDC** — the publish `PUT` 404s (`'<pkg>@<version>' is not in this registry`)
  even with a correct Trusted Publisher configured. The signature is: provenance
  signs, then 404 on PUT. `publish.yml` runs `npm install -g npm@latest` after
  `setup-node` for exactly this reason; do not remove it.
- **Trusted Publishers are configured per-package on npmjs.com**, not in the repo.
  `@tasks-md/parser`, `@tasks-md/lint`, `@tasks-md/cli`, and `tasks-mcp` each list
  `tasksmd/tasks.md` -> `publish.yml` (no environment). All four are set.
- **`scripts/sync-versions.sh` must skip the private `@tasks-md/conformance`.**
  Bumping its cross-reference to `^<version>` makes `npm ci` try to fetch the
  unpublished package from the registry -> 404. It stays pinned `*` so it always
  resolves to the local workspace.
- **CI's `npm ci` resolves through Intuit Artifactory**, which times out
  (`ETIMEDOUT`) on brand-new dependency versions that aren't mirrored yet (a fresh
  `yaml@2.9.0` broke CI this way — the workspaces config is now dependency-free
  JSON). Prefer mature, already-mirrored dependency versions; trust the GitHub
  Actions run over a local `npm view` (the Artifactory mirror lags the public registry).
- **`tasks-claim-check` is advisory by default** (warns, never blocks, so it never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tasksmd/tasks.md](https://github.com/tasksmd/tasks.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
