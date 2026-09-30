---
trigger: always_on
description: > **This directory is part of [labsai/EDDI](https://github.com/labsai/EDDI).** It was the separate `labsai/EDDI-Manager` repository until 2026-09-15; its full history was imported here (`git log -- ui/manager`). Issues and pull requests go to `labsai/EDDI`. The UI is built into the EDDI jar by Maven from the repository root — see the root `AGENTS.md` (Build & Test Commands).
---

# EDDI Manager — AI Agent Instructions

> **This directory is part of [labsai/EDDI](https://github.com/labsai/EDDI).** It was the separate `labsai/EDDI-Manager` repository until 2026-09-15; its full history was imported here (`git log -- ui/manager`). Issues and pull requests go to `labsai/EDDI`. The UI is built into the EDDI jar by Maven from the repository root — see the root `AGENTS.md` (Build & Test Commands).

> **This file is automatically loaded by AI coding assistants. Follow ALL rules below.** The [root `AGENTS.md`](../../AGENTS.md) applies here too — branching, push approval, commit attribution and the changelog rule are defined there once and not repeated below.

## 1. Project Context

**EDDI Manager** is the admin dashboard for the [EDDI](https://github.com/labsai/EDDI) conversational AI platform. It is a **React/TypeScript SPA** served from the EDDI backend at `/manage`.

### Ecosystem

The Manager, the Chat UI and the backend are one repository, `labsai/EDDI`:

| Location | Tech | Purpose |
| --- | --- | --- |
| **repo root** | Java 25, Quarkus, MongoDB or PostgreSQL | Backend engine, REST API, lifecycle pipeline, integration tests (`src/test/java/**/*IT.java`) |
| **`ui/manager`** (this) | React 19, Vite, Tailwind | Admin dashboard — agents, workflows, extensions, chat |
| **`ui/chat`** | React, TypeScript | Standalone chat widget |
| **eddi-website** | Astro | Marketing site at eddi.labs.ai |

### Tech Stack

| Layer | Technology |
| --- | --- |
| **Build** | Vite 8 |
| **UI** | React 19 + TypeScript 5 (strict) |
| **Styling** | Tailwind CSS v4 + CSS variables (black/gold) |
| **State (server)** | TanStack Query v5 |
| **State (UI)** | Zustand (chat/debug), `useState` / `useCallback` elsewhere |
| **Routing** | React Router v7 (`react-router-dom` 7.x, declarative mode — no data router) |
| **i18n** | react-i18next (11 locales: en, de, fr, es, ar, zh, th, ja, ko, pt, hi) |
| **Test (unit)** | Vitest + React Testing Library + MSW |
| **Test (e2e)** | Playwright |
| **Editor** | Monaco (@monaco-editor/react) |
| **DnD** | @dnd-kit (workflow pipeline builder) |

---

## 2. Workflow

### Before Starting Any Work

1. **Check state**: `git status`, `git log -5 --oneline`, `git branch --show-current`
2. **Recent context**: the top entries of the root [`docs/changelog.md`](../../docs/changelog.md) and anything pending in [`docs/changelog.d/`](../../docs/changelog.d/README.md) — that is where Manager work is recorded (root rule 8)
3. **Backend context**: the [root `AGENTS.md`](../../AGENTS.md) when touching API contracts
4. **[`HANDOFF.md`](HANDOFF.md)** is the Manager's running log from before the monorepo move — about 180 KB. **Do not read it end to end.** Search it for the screen or feature you are touching; its deep dives (the operator, secrets grants, the Workforce decisions) are worth finding when you need them

### During Work

- **Branch** per root rule 3 — never commit to `main`, branch from `origin/main`, name it `feat/…`/`fix/…` (never a tool-generated `claude/…` name).
- **Commit often** with conventional commits scoped to the area: `feat(manager): …`, `fix(manager): …`, or a feature scope such as `fix(operator): …`. Use `(ui)` only for a change that spans both UIs.
- **Record the change** in a new `docs/changelog.d/YYYY-MM-DD-<slug>.md` fragment at the repository root (root rule 8), in the same commit as the work. Do not add entries to `HANDOFF.md` — every PR editing the same file is the merge conflict the fragments exist to avoid.

### ⚠️ Dependency changes on Windows break CI's `npm ci`

`@tailwindcss/oxide-wasm32-wasi` is an optional package that Windows skips, so npm
on Windows never resolves its children and **prunes them from
`package-lock.json`** on any `npm install` / `npm uninstall`. The Linux CI runner
then fails before it runs anything:

```
npm error `npm ci` can only install packages when your package.json and
npm error package-lock.json are in sync.
npm error Missing: @emnapi/core@1.11.3 from lock file
```

Neither `npm install --package-lock-only` nor `--os=linux --cpu=x64` re-adds them.
After changing any dependency on Windows, check the lock still carries all four:

```bash
node -e "const l=require('./package-lock.json');Object.keys(l.packages).filter(k=>k.includes('emnapi')||k.includes('wasm-runtime')).forEach(k=>console.log(k,l.packages[k].version))"
```

Expect `@emnapi/core`, `@emnapi/runtime`, `@emnapi/wasi-threads` and
`@napi-rs/wasm-runtime` nested under
`node_modules/@tailwindcss/oxide-wasm32-wasi/node_modules/`. If any are gone,
restore them from the last lockfile CI accepted rather than regenerating.

### Quality Gates

There is no pre-commit hook (the husky + lint-staged hook did not survive the move into the
EDDI monorepo). CI's `UI Manager Checks` job runs these on every PR into `main` that touches `ui/`
(`ci.yml` runs on no other base branch), in this order — run them yourself before pushing:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [labsai/EDDI](https://github.com/labsai/EDDI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
