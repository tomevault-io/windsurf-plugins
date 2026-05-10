---
trigger: always_on
description: If `README.md` exists at the repo root, read it first.
---

If `README.md` exists at the repo root, read it first.

## Ecosystem

All projects with lintmax in deps are managed by pm4ai (`bunx pm4ai@latest`). The tool syncs configs, generates CLAUDE.md, enforces conventions, and runs maintenance.

Key repos:

- **pm4ai** — the management tool. Rules in `apps/docs/content/rules/*.mdx`. Checks in `packages/pm4ai/src/`.
- **lintmax** — max-strict lint/format orchestrator. All projects depend on it.
- **cnsync** — canonical source for `readonly/ui` (shadcn + ai-elements components).

For full documentation: `curl https://pm4ai.vercel.app/llms-full.txt`

## Managed Files

These files are auto-generated and synced by pm4ai. Never edit them directly:

- `CLAUDE.md` — generated from rules based on project dependencies
- `.github/workflows/ci.yml` — universal CI workflow
- `clean.sh` — universal cleanup script
- `up.sh` — universal maintenance cycle (clean + install + build + fix + check)
- `bunfig.toml` — bun configuration
- `.gitignore` — universal ignore patterns
- `readonly/ui/` — synced from cnsync

## Role Detection

Run `gh auth status` to determine your role:

- If you are the repo owner (`1qh`): you can modify pm4ai rules and checks directly in the pm4ai repo
- Otherwise: do not edit managed files, only use companion files

## Owner Workflow

- `bunx pm4ai@latest status` — check current project
- `bunx pm4ai@latest fix` — sync + maintain (requires clean git)
- `bunx pm4ai@latest fix --all` — all projects
- New universal rule → add `.mdx` to pm4ai `apps/docs/content/rules/` with `infer` frontmatter
- New check → add to pm4ai `packages/pm4ai/src/audit.ts` or `checks.ts`
- If you discover something during this session that should apply to all projects, note it for pm4ai

## Companion Files

Use these for project-specific content:

- `LEARNING.md` — lessons learned, gotchas, known issues
- `RULES.md` — project-specific rules that don’t apply to other projects
- `PROGRESS.md` — tracking ongoing work
- `PLAN.md` — planning and architecture decisions

## Health Check

Run `bunx pm4ai@latest status` to see issues in this project. Run `bunx pm4ai@latest fix` to sync and maintain.

Check states:

- `check: passed 5m ago (current)` — safe to proceed
- `check: passed 3h ago (before 2 commits)` — stale, run `bunx pm4ai@latest status` again and wait for refresh
- `check: failed 5m ago (current), 15 violations` — fix violations before proceeding
- `check: running...` — wait, do not edit files until complete
- `check: never run` — run `bunx pm4ai@latest fix` first

---

## Package Manager

- Only `bun` — yarn/npm/npx/pnpm forbidden
- `bun fix` must always pass
- Never `bun update` — it replaces `"latest"` with resolved versions
- Always `bun clean && bun i` to update deps
- All deps use `"latest"` tag — no pinned versions unless necessary
- For deps that must be pinned, pin major version only (e.g. `"eslint": "9"`)
- No lockfile committed (`bun.lock` in `.gitignore`)

## Bun APIs

- Always `import { X } from 'bun'` — never use `Bun.X` global (triggers `noUndeclaredVariables` in biome)
- `import { $, file, write, Glob, spawn } from 'bun'`

## Scripts

- `sh clean.sh` — nuke all artifacts (node_modules, lockfile, caches, dist, .next)
- `sh up.sh` — clean + install + fix + check (the universal maintenance cycle)
- Scripts: silent on success, verbose on failure
- Never use `git clean` — it deletes `.env` and uncommitted files. Use explicit `rm -rf`.

---

## Must NOT Do

- NEVER write comments (lint ignores allowed)
- NEVER touch `readonly/ui/` manually
- NEVER use `!` (non-null assertion), `any`, `as any`, `@ts-ignore`, `@ts-expect-error`
- NEVER duplicate types — single source of truth
- NEVER disable lint rules globally/per-directory — fix the code
- NEVER ignore written source code from linters — only auto-generated code (`_generated/`, `generated/`, `module_bindings/`, `readonly/ui/`)
- NEVER reduce lintmax strictness — if upstream removes rules, find replacements

## Type Safety & Single Source of Truth

Every piece of data flows through exactly one definition:

- Shared constants → define once, import everywhere. If a value appears in 2+ files, extract it.
- When adding new features, check existing utilities FIRST before writing inline logic.

## Code Consolidation Checklist

Before writing any new code, verify:

1. Does this function already exist? Check existing utilities first
2. Is this constant defined elsewhere? Check shared files
3. Am I adding a wrapper div? Check if parent `gap-*`, `space-*` can handle it
4. Am I adding inline styles? Only allowed for truly dynamic values. NEVER for colors or static properties.
5. Am I copy-pasting from another file? Extract to a shared utility/component

---

## Commits

- Commit frequently, push logical groups
- No AI tooling in commits
- Format: `type: description` (fix, feat, docs, chore, refactor, test)

---

## Lintmax

lintmax combines biome, oxlint, eslint, prettier, and sort-package-json into one command. We own it.

### CLI — agents MUST read this

`lintmax fix` fixes everything, then runs a full verification internally (all 5 linters run twice — once to fix, once to check). Silent on success. Exit 0 = fully clean.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1qh/idecn](https://github.com/1qh/idecn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
