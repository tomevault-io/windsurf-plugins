---
trigger: always_on
description: You are working on **kandown itself**: a file-based Kanban engine backed by plain
---

# Agent instructions

You are working on **kandown itself**: a file-based Kanban engine backed by plain
Markdown, installed into other projects with `npx kandown init`.

*(If you are working on a project that merely **uses** kandown, you want
`kandown work`, not this file.)*

---

## Read this in order

| # | Read | When |
|---|---|---|
| 1 | **This file** | Always: the hard rules are below |
| 2 | [`CODEMAP.md`](CODEMAP.md) | To find the file that owns a concern |
| 3 | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Before any non-trivial change |
| 4 | `kandown work` | Any task, backlog or board work |
| 5 | [`docs/RELEASE.md`](docs/RELEASE.md) | On "bump" / cutting a release |
| 6 | [`README.md`](README.md) | For the user-facing feature surface |
| 7 | [`docs/EXTENSIONS.md`](docs/EXTENSIONS.md) | Before touching the extension/plugin system, or anything under `plugins.*` |
| 8 | [`docs/EXTENSIONS-AUTHORING.md`](docs/EXTENSIONS-AUTHORING.md) | When building or scaffolding an extension |
| 9 | [`docs/EXTENSIONS-AGENT.md`](docs/EXTENSIONS-AGENT.md) | When *writing* a plugin: the generated contract, also printed by `kandown plugin brief` |

Then run **`kandown work`**: it prints the current agent rules plus a live board
digest and the next actionable task.

Look a symbol up in `CODEMAP.json` instead of grepping. Ask what depends on what
with `graphify query "<question>"` (see
[Architecture § the dependency graph](docs/ARCHITECTURE.md#the-dependency-graph)).

## Change fan-out checklist

Some interfaces intentionally mirror the same feature. Before editing one file
in a row, inspect every surface listed and keep policy in a shared module.

| Concern | Surfaces to inspect | Shared owner |
|---|---|---|
| Task editor UI | `TaskWorkspace.tsx` (desktop), `Drawer.tsx` (mobile) | shared components such as `TaskExtensionSurface.tsx` |
| Zustand board actions | `src/lib/store.ts`, `src/lib/store/boardSlice.ts` | filesystem and domain helpers under `src/lib/` |
| Local API behavior | `src/cli/lib/server.ts`, `vite.config.ts`, `src/lib/demoBackend.ts` | shared coordinators under `src/cli/lib/` or `src/lib/` |
| Generated outputs | `bin/*`, `.kandown/kandown.html`, `CODEMAP.*`, `CHANGELOG.md` | source, build scripts and release files listed below |

The detailed rationale lives in
[Architecture § interface adapters and mirrors](docs/ARCHITECTURE.md#interface-adapters-and-mirrors).

## Review authority

Vava's explicit decisions and the active task's `Decisions` / `Out of scope`
sections are authoritative. A reviewer may find bugs inside that contract, but
must not silently turn an accepted risk or deferred feature into a blocker. List
such observations separately as non-blocking follow-up work. Ask vava before
expanding scope, changing a settled architecture decision or reverting verified
behavior.

---

## Hard rules

### 1. Never edit a generated file

They are committed because they ship. Editing them appears to work, and the next
`pnpm build` silently erases it.

| Never edit | Edit instead |
|---|---|
| `bin/kandown.js` | `src/cli/cli.ts` |
| `bin/tui.js` | `src/cli/tui.tsx` |
| `src/lib/version.ts` | `package.json` (version field) |
| `CODEMAP.md`, `CODEMAP.json` | the JSDoc headers, then `pnpm codemap` |
| `CHANGELOG.md` | `changelogs/v<version>.md`, then `pnpm changelog` |
| `docs/EXTENSIONS-AGENT.md`, `src/lib/extensions/agent-brief.ts` | `src/lib/extensions/types.ts` or the prose in `scripts/build-extension-brief.js`, then `pnpm extension-brief` |

`CODEMAP.md` flags each of these inline, at the moment you go looking for it.

### 2. Every source file carries a JSDoc header

`@file`, `@description`, and `@functions` / `@exports` where they help. This is not
decoration: `CODEMAP.md` is built from these headers, and `pnpm codemap:check` (run
in CI) **fails** when a file has no `@description`. Coverage is at 100%; keep it
there.

Write the header *after* the feature works, not before. Explain **why** and **when
it runs**, not just what it is. Use `📖` to open explanatory comments, matching the
surrounding files.

### 3. Keep kandown tasks up to date, as you work

Not at the end. Check off subtasks with a `report:` line as you complete them, move
the task to the right column, and write a real completion report. `kandown work`
has the full protocol. The task file *is* the work log; if the user opens it, it
should show exactly where things stand.

### 4. UI text is authored in English

English is the source of truth in `src/lib/i18n/locales/`. Translate *from* it,
never into it. "Translate all" means: diff each locale against English, fill the
missing keys, repeat for every language.

### 5. One changelog file per release, and name the release yourself

On a **bump**, the release notes go in a **new file**: `changelogs/v<version>.md`,
headed exactly

```markdown
# <version> - <YYYY-MM-DD> - "<name>"
```

Never append to `CHANGELOG.md`: it is a generated index, rebuilt from
`changelogs/` by `pnpm changelog` and staged automatically by the pre-commit
hook. CI runs `pnpm changelog:check` and fails on drift.

**Choose the release name yourself and do not ask for confirmation.** One to
three words describing the largest change ("Two Views", "TUI Agents", "Motion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vava-nessa/kandown](https://github.com/vava-nessa/kandown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
