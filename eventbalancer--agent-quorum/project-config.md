---
trigger: always_on
description: `agent-quorum` is a standalone TypeScript CLI and library for the iterative
---

# agent-quorum — Agent Operating Rules

`agent-quorum` is a standalone TypeScript CLI and library for the iterative
plan -> critique -> update loop across Codex, Claude Code, and Cursor Agent
CLIs.

Global development conventions live in
[`docs/development/conventions.md`](docs/development/conventions.md). This file
keeps the operating rules that agents should load before changing this
repository.

## 1. Hard Constraints

Override all other project-level guidance.

- **English everywhere in committed artifacts.** Code, comments, commits, docs,
  tests, config, schemas, and prompts are written in English. User-facing
  conversation can be Russian when the user uses Russian.
- **Resolved paths in conversation.** When reporting paths to the user, use
  resolved absolute paths for this machine. Keep portable forms such as
  repo-relative paths and `$HOME` for committed files and reusable snippets.
- **Never commit or push without explicit user instruction.**
- **No destructive git or shell operations.** Do not run force pushes,
  `git reset --hard`, broad `git restore`, `git clean`, `rm -rf`, `sudo`, or
  permission-recursive commands unless the user explicitly asks and the risk is
  clear.
- **Use repo entry points.** Prefer `pnpm run <script>` and `pnpm exec <bin>`;
  never use `npx`.
- **Do not hand-edit generated artifacts.** `dist/`, `coverage/`, lockfiles, and
  package-manager output are generated through the project commands.
- **Preserve the public API.** Keep `src/index.ts`, `package.json` exports, and
  the `agent-quorum` bin stable unless a breaking change is explicit.
- **Respect architecture boundaries.** Provider calls go through
  `providerRun`; `core/` does not spawn provider CLIs directly, and lower layers
  do not import higher layers.
- **Treat role skills and schemas as contracts.** Files under `skills/` define
  provider I/O behavior. Changing a schema or role prompt changes the runtime
  contract and needs matching tests or documentation.
- **Source comments are exceptional.** Prefer names, types, tests, and structure
  over comments. Add comments only for critical non-obvious invariants, specific
  external bugs or provider quirks, or behavior that names and types cannot
  express. Do not restate what the code already says; describe the current task,
  branch, or PR; leave TODO/FIXME/HACK breadcrumbs; duplicate a function
  signature; or preserve commented-out code. If a block needs a comment to
  explain what it does, extract a named helper or boolean instead. Public API
  docblocks may explain invariants, units, failure modes, and external contracts;
  do not write parameter-by-parameter docblocks that repeat the type signature.
- **Dogfood through the real CLI.** For changes that should be designed by
  `agent-quorum` itself, drive the loop with `pnpm run plan:self -- --prompt …`
  (the `agent-quorum` bin from source, no build required); see
  [`examples/`](examples/).
- **Isolate sessions in worktrees.** Run nontrivial, multi-file, or potentially
  concurrent work in a session worktree
  (`pnpm run worktree:create <slug> --desc <text>`), not the shared checkout;
  integrate to `main` via `/ship` plus an explicit step. See
  [Session Worktrees](docs/development/conventions.md#session-worktrees).
- **No orphan background shells.** Do not leave long-running shell sessions or
  detached commands alive after moving on.

## 2. Sources of Truth

When facts conflict, trust in this order:

1. This file, mirrored by the `AGENTS.md` symlink.
2. [`docs/development/conventions.md`](docs/development/conventions.md) for
   code, git, verification, and style rules.
3. `eslint.config.ts`, `tsconfig.json`, and `vitest.config.ts` for enforced
   tooling behavior.
4. `package.json` for scripts, package exports, bin entries, engines, and direct
   dependencies.
5. [`docs/architecture.md`](docs/architecture.md),
   [`docs/configuration.md`](docs/configuration.md), [`docs/cli.md`](docs/cli.md),
   and [`docs/api.md`](docs/api.md) for runtime contracts.
6. `src/core/defaults.ts` (built-in orchestration defaults, mirrored by
   `config.example.json`) and `skills/**/*.schema.json` for default
   orchestration and role I/O contracts.

## 3. Required Entry Points

| Task                       | Use                                                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Install dependencies       | `pnpm install --frozen-lockfile`                                                                                           |
| Build                      | `pnpm run build`                                                                                                           |
| Typecheck                  | `pnpm run typecheck`                                                                                                       |
| Lint                       | `pnpm run lint`                                                                                                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eventbalancer/agent-quorum](https://github.com/eventbalancer/agent-quorum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
