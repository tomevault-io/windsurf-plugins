---
trigger: always_on
description: This repository is a **pnpm monorepo**. Read [CONTRIBUTING.md](./CONTRIBUTING.md)
---

# AGENTS.md

This repository is a **pnpm monorepo**. Read [CONTRIBUTING.md](./CONTRIBUTING.md)
and [docs/DEVELOPMENT.md](./docs/DEVELOPMENT.md) before changing code.

## Where to document things

- **`AGENTS.md`** is for agent-facing operating rules that should be loaded
  before work starts. Keep it short and high-signal.
- **`docs/DEVELOPMENT.md`** is the canonical engineering guide: package
  boundaries, naming, UI conventions, tests, local board workflow, and
  "what to update when behavior changes".
- **`RELEASING.md`** is the canonical release/deploy runbook for CLI + Desktop.
- **`docs/TROUBLESHOOTING.md`** is for known user/operator failure modes.
- **`docs/ROADMAP.md`** is strategy, not shipped behavior.
- Do **not** add an `AGENTS.md` to every package by default. Add a nested
  `AGENTS.md` only when a subtree has rules that contradict or specialize
  this file.

## Layout (canonical)

- `packages/cli/` — `backlog` (Apache-2.0, npm: [`backlog`](https://www.npmjs.com/package/backlog))
- `packages/desktop/` — Electron app (`@backlog/desktop`, private package, GitHub Releases)
- `packages/board-ui/` — Svelte board shared by CLI serve and Desktop
- `packages/server/` — local Hono API server used by CLI serve and Desktop
- `packages/{core,claims,connectors,config,git,hooks,schemas}/` — workspace-internal modules, bundled into the CLI tarball at publish time
- `docs/ROADMAP.md` — multi-target roadmap (sources, repositories, sandboxes, executors, deploy targets)
- Root `README.md` is symlinked from `packages/cli/README.md` (CLI README is the canonical one for npm)

## Package boundaries

- `packages/schemas/` (Zod) is the **source of truth for cross-boundary types**.
- Internal packages use `workspace:*` deps; tsup bundles everything for the published `backlog` tarball.
- The cloud backend (Backlog Cloud) lives in a private repository and is not part of this monorepo.

## Naming conventions

- User-facing copy says **project**, **repository/repositories**, **task**, **subtask**,
  **run**, **claim**.
- Do not use "repo" or "repos" in new product copy or new public-facing code
  names. Keep `repo`/`repos` only for backward-compatible API fields, CLI
  flags, route names, storage keys, and legacy symbols until a planned
  compatibility migration replaces them with `repository`/`repositories`.
- Do not introduce new user-facing "workspace" copy. "Workspace" may still
  appear in legacy/internal code when the storage concept is literally
  `.backlog/`; prefer "project" for new public API and UI.
- Do not reintroduce the legacy task terminology in UI, docs, API names,
  tests, or new code. Use task/subtask. If touching legacy symbols, rename
  them when practical and low-risk.
- Local users are local assignable people. Real email invitations and shared
  multi-user collaboration belong to Backlog Cloud.

## Common commands

```sh
pnpm install                    # install workspace
pnpm test                       # vitest run (workspace-wide)
pnpm typecheck                  # tsc --noEmit per package
pnpm --filter @backlog/board-ui typecheck
pnpm --filter backlog dev       # CLI dev mode (tsx)
pnpm --filter backlog build     # CLI build (tsup)
pnpm --filter @backlog/desktop build
pnpm --filter backlog pack:check
```

Run the local board from the repository root:

```sh
pnpm --filter backlog dev serve --project /Users/jimmy/Dev/backlog/backlog-cli --port 7878
```

If the UI says "API ready, UI bundle missing", build the board first:

```sh
pnpm --filter @backlog/board-ui build
```

## CLI conventions

- Top-level canonical commands: `init`, `doctor`, `update` (no `setup` prefix).
- Grouped namespaces for the rest: `repos`, `work`, `task`, `claim`, `hooks`, `schedule`, `runs`, `agents`, `sources`, `release`, `worktree`.
- Version flag: `-v, --version` (lowercase).
- Default mode is fully local. Cloud sync (when available) is opt-in.

## Board/UI conventions

- The board UI is Svelte 5 in `packages/board-ui`.
- Keep operational screens dense and tool-like; avoid marketing layouts inside
  the app.
- Use existing i18n keys in `packages/board-ui/src/lib/i18n/{en,fr}.json`.
  Any visible copy change needs both languages.
- Repository removal in the UI means "remove from Backlog", not delete files or
  cascade through tasks. Dangerous/destructive Git operations need explicit,
  precise confirmation.
- If a backend route changes behavior, update the corresponding UI API wrapper
  in `packages/board-ui/src/lib/api.ts` and add/adjust route tests.

## Workspace location convention

Two layouts, both seeded by `backlog init`:

- **`in_repo`** (default) — workspace at `<project root>/.backlog/`. Right
  for single-repo projects.
- **`user_level`** (opt-in via `--user-level`) — workspace at
  `~/.backlog/<slug>/`. Right for multi-repo projects so the workspace
  isn't tied to one repo. `<slug>` must not collide with another
  registered user-level project.

Both layouts share the same internal file shape (`config.toml` plus the
YAML/JSON state files). The choice is recorded in `config.toml` as
`project_location`, mirrored in the user registry's per-entry `location`
field.

The user registry (`projects.json`) lives at `~/.backlog/projects.json` on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osmove/backlog](https://github.com/osmove/backlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
