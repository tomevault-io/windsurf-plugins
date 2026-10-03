---
trigger: always_on
description: Guidance for anyone — human or agent — working in this codebase.
---

# AGENTS.md

Guidance for anyone — human or agent — working in this codebase.

This file is deliberately **thin**: the rules that always apply, plus a table
routing to deep-dives. Load the one matching what you are working on rather than
carrying all of it every turn.

## Project overview

The Hive is a **command center for multiple agentic terminal sessions** running on
a single machine. An orchestrator ("Concierge"-style coordinator, called *maestro*
in the console) routes messages between the user and sessions, surfaces questions
and permission requests as an inbox, tracks PRs and tickets, and spawns new
sessions. **The most important component is the embedded terminal at the center of
the screen** — everything else exists to route the user's attention to the right
terminal at the right moment.

Current phase: terminals are **real PTYs**, projects come from a config file,
tickets from **Jira**, PRs from `gh`, and notifications from Claude Code's hooks.
The right rail's third tab is a **project explorer** over the active session's
repository, opening files into a CodeMirror editor on the centre stage. Full
context and scope: the **HIVE project in Jira**, the backlog.

Stack: React 19 · TypeScript (strict) · Vite · xterm.js · CodeMirror 6 ·
Zustand · Tailwind v4 · shadcn/ui · pnpm.

## Essential commands

| Command | What it does |
| --- | --- |
| `pnpm dev` | Vite dev server — the **browser** target (chrome only: no PTYs, no Jira, no config) |
| `pnpm build` | Type-check, then production build of the browser target |
| `pnpm desktop:dev` | electron-vite: the Electron app, renderer HMR included |
| `pnpm desktop:build` | Type-check, then build `out/{main,preload,renderer}/` |
| `pnpm desktop:preview` | Run the built Electron app |
| `pnpm desktop:dist` | Package `.dmg` + `.zip` into `dist/` (macOS arm64); `:publish` uploads them |
| `pnpm lint` | ESLint across `src/`, `electron/` and config |
| `pnpm type-check` | `tsc --noEmit` for the app, the Node-side configs, and `electron/` |
| `pnpm test` | Vitest, single run |
| `pnpm test:coverage` | Vitest with the 80% coverage gate |
| `pnpm test:e2e` | Playwright — both the web and electron projects |
| `pnpm test:e2e:web` · `:electron` | Either half alone — browser specs (070), or the built app (085) |
| `pnpm test:pty` | PTY conformance — real PTYs, Electron ABI, no UI (098) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yunidbauza/the-hive](https://github.com/yunidbauza/the-hive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
