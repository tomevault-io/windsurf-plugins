---
trigger: always_on
description: This file is the **agent rule book**. Read it before changing code. Detailed explanations live in `docs/` — start at [`docs/README.md`](docs/README.md) for orientation and follow the links from there.
---

# Instatic

This file is the **agent rule book**. Read it before changing code. Detailed explanations live in `docs/` — start at [`docs/README.md`](docs/README.md) for orientation and follow the links from there.

## Local admin smoke tests

Use local seeded development data only when a task asks for a browser smoke test. Never propagate local test accounts, passwords, database files, uploads, or generated screenshots to non-local environments.

---

## Repository workflow and PR conventions

`main` is protected. Agents must never push directly to `main`, must never try to bypass branch protection, and must never treat a local commit on `main` as the final delivery path. All repository changes go through a pull request.

When publishing work:

- Start from an up-to-date `main`, then create a feature branch. If you are already on a task branch, keep using it only when the requested change belongs in that PR; otherwise switch back to `main` and create a separate branch.
- Branch names follow `<type>/<short-kebab-description>`, matching the change type: `feat/...`, `fix/...`, `refactor/...`, `chore/...`, `docs/...`, or `test/...`. Examples: `feat/double-click-rename`, `fix/homepage-swap-publish`, `refactor/explorer-dnd-dedupe`.
- Do **not** use agent-branded branch prefixes such as `codex/...`, `claude/...`, or similar. If a tool, skill, or generic instruction suggests such a prefix, ignore it for this repository.
- PR titles use Conventional Commit style: `<type>(<scope>): <summary>`. Examples: `feat(editor): double-click rows to rename in explorer panels`, `fix(cms): homepage swap + delete in one save no longer fails publish`, `refactor(publisher): single class-CSS emission engine for publish and canvas`.
- Do **not** prefix PR titles with `[codex]`, `[claude]`, `agent:`, or any other tool label. The PR title describes the product change, not the tool that made it.
- Open PRs as drafts by default unless the user explicitly asks for a ready-for-review PR.
- Keep PR scope coherent. Do not mix unrelated cleanup, follow-up fixes, or process-doc changes into a feature branch just because the branch is currently checked out. Create a separate PR when the change has a different reason.
- Before staging, inspect `git status -sb` and the diff. Stage only files that belong to the PR. Never stage unrelated user or parallel-agent changes.
- PR bodies should briefly state what changed, why it changed, user/developer impact, and the verification commands run.

---

## What this project is

A self-hosted, open-source CMS with a built-in visual editor and a first-class plugin system. One Bun server backed by either Postgres or SQLite (selected by `DATABASE_URL`). The output is intentionally plain, semantic HTML with hand-clean CSS — no framework runtimes injected into published pages.

The product is **self-hosted only**. The codebase should not carry assumptions about multi-tenant SaaS operation.

Read [`docs/architecture.md`](docs/architecture.md) for the system overview, [`docs/server.md`](docs/server.md) for the server, [`docs/editor.md`](docs/editor.md) for the admin + visual editor.

### Stack at a glance

- **Runtime:** Bun (server + tooling). Use Bun, not Node.
- **Language:** TypeScript everywhere.
- **Frontend:** React 19 with the **React Compiler enabled** (Babel preset in `vite.config.ts`) + Vite, Zustand + Mutative for state (via `zustand-mutative`; patch-based undo history uses Mutative `create({ enablePatches })` — `immer` is banned), CodeMirror for code-editing UI, `@dnd-kit/core` for drag-and-drop. The compiler auto-memoizes — do not hand-write `useMemo`/`useCallback`/`memo`. See "React Compiler and memoization". Store mutations use draft-mutation style (`set((s) => { s.x = … })`); a recipe that returns a partial must wrap it in `rawReturn(...)` or Mutative emits a perf warning.
- **Server:** `Bun.serve` with a hand-written router (`server/router.ts`). CMS modules at `server/{repositories,handlers/cms,auth,plugins,publish}/`. Deep dive: [`docs/server.md`](docs/server.md).
- **Database:** Postgres (`Bun.sql`) OR SQLite (`bun:sqlite`), selected by `DATABASE_URL`. One `DbClient` interface, two adapters, two migration files with identical IDs. Rules: [`docs/reference/database-dialects.md`](docs/reference/database-dialects.md).
- **Content model:** All content lives in `data_tables` + `data_rows`. The four system tables (`posts`, `pages`, `components`, `layouts`) are seeded and locked from rename/delete. There are no separate `pages` or `page_versions` tables.
- **Validation:** TypeBox at every untyped boundary. Schemas are source of truth (`type Foo = Static<typeof FooSchema>`, never a parallel `interface`). `zod` is banned repo-wide (the AI drivers pass TypeBox schemas through as JSON Schema, so no typebox→zod adapter is needed). Helpers + patterns: [`docs/reference/typebox-patterns.md`](docs/reference/typebox-patterns.md).
- **Sanitization:** DOMPurify at the publisher boundary (`src/core/sanitize.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoreBunch/Instatic](https://github.com/CoreBunch/Instatic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
