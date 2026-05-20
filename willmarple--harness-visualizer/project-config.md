---
trigger: always_on
description: > **This is the primary source of truth for working on the Harness Visualizer.**
---

# AGENTS.md

> **This is the primary source of truth for working on the Harness Visualizer.**
> Any agent — Claude Code, Cursor, Copilot, Aider, Windsurf, Codex, anything — should read this file in full before reading any other file in the repo, and before making any edits. Tool-specific configs (`CLAUDE.md`, `.cursor/rules/`, etc.) point back to this file by design.

---

## What this project is

A tool-agnostic, real-time **harness audit and visualization** demo app. Built as a conference-talk demo for harness engineering and dogfooding.

The app:

1. Watches a user-selected directory tree.
2. Finds harness artifacts (context files, skills, MCP configs, hooks, subagents) using a configurable pattern map.
3. Renders them as an interactive graph in the browser, color-coded by layer.
4. Lets the user click any markdown node to view/edit it in-browser; saves write back to disk; the file watcher closes the loop and the graph re-renders live.

The "wow moment" for the talk is **point the tool at its own repo, edit a skill file in the browser, watch the graph update in real time.** Everything else serves that moment.

Long-form context lives in [`harness-visualizer-plan.md`](./harness-visualizer-plan.md) (the deep-research seed doc) and [`ROADMAP.md`](./ROADMAP.md) (the phase-by-phase build plan). Read both before designing or implementing anything substantial.

---

## Stack

### Backend — Node.js + TypeScript

| Package | Role |
|---|---|
| `express` | HTTP server, REST endpoints (`/api/browse`, `/api/file`, `/api/patterns`) |
| `socket.io` | Real-time push to frontend (`harness:update`, `configure-watch`, `patterns:update`) |
| `chokidar` | File watching, supports `.add()`/`.unwatch()` for live reconfiguration |
| `fast-glob` | Initial directory scan (faster than chokidar's initial pass on large trees) |
| `gray-matter` | YAML frontmatter parser for skill files, rules, AGENTS.md, etc. |
| `zod` | Schema validation for artifacts, IPC payloads, and request bodies (re-exported via `@harness-visualizer/shared`) |
| `vitest` | Test runner |

### Frontend — Vue 3 + Vite + TypeScript

| Package | Role |
|---|---|
| `pinia` | State management — Pinia setup-stores hold `harness:update` payload state |
| `socket.io-client` | WebSocket client (paired with reactive Pinia state); typed `Socket<S2C, C2S>` (note flipped order vs server) |
| `v-network-graph` | SVG graph rendering, drag/pan/zoom, `node:click` events. Consumes `Record<string, T>` (id-keyed), NOT arrays. Phase 3 ships a `computed()` adapter from `Node[]/Edge[]` |
| `d3-force` | Peer-dep of `v-network-graph` (force-directed layout via `v-network-graph/lib/force-layout`); install explicitly |
| `@vueuse/core` | Composables — `useDark` for theme toggle |
| `tailwindcss` v4 | Utility-first styling, CSS-first config (`@import "tailwindcss"` + `@theme` + `@custom-variant dark`); paired with `@tailwindcss/vite` plugin |
| `md-editor-v3` | Markdown viewer + editor (`MdEditor`, `MdPreview`, `MdCatalog`) — Phase 5 |
| `vitest` + `@vue/test-utils` | Component + unit tests; mock `socket.io-client` and stub `v-network-graph` in `frontend/src/test/setup.ts` |

### Tooling

- npm **workspaces** (root `package.json` lists `backend`, `frontend`, `shared`).
- TypeScript shared base config at `tsconfig.base.json`, extended per-package.
- ESLint flat config + Prettier at the repo root.
- `concurrently` for the root `dev` script that boots both servers.
- **pnpm migration trigger**: npm workspaces is sufficient for ≤3 packages without hoisting conflicts. Revisit pnpm if a fourth workspace lands or hoisting conflicts emerge.

---

## Repo layout

```
harness-visualizer/
├── AGENTS.md                            ← you are here (source of truth)
├── CLAUDE.md                            ← thin pointer to AGENTS.md
├── ROADMAP.md                           ← 8-phase build plan; one spec per phase
├── harness-visualizer-plan.md           ← deep-research seed doc
├── package.json                         ← workspaces, root scripts
├── tsconfig.base.json
├── eslint.config.js                     ← flat config
├── .prettierrc
│
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── server.ts                    ← Express + Socket.IO entrypoint
│       ├── patterns/                    ← default pattern map + loader
│       ├── scanner/                     ← fast-glob + gray-matter + zod
│       ├── watcher/                     ← chokidar wrapper + hand-rolled debounce
│       ├── scoring/                     ← per-layer presence/validity scores
│       ├── security/                    ← validatePath helper (Vite-style isParentDirectory)
│       └── __test-helpers/              ← withTmpRoot for integration tests
│
├── shared/                              ← @harness-visualizer/shared workspace
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── schemas.ts                   ← zod runtime schemas (PatternMap, Diagnostic, HarnessUpdatePayload, etc.)
│       ├── types.ts                     ← z.infer-derived TS types
│       ├── events.ts                    ← Socket.IO ServerToClientEvents / ClientToServerEvents
│       └── index.ts                     ← named re-exports
│
├── frontend/
│   ├── package.json
│   ├── vite.config.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willmarple/harness-visualizer](https://github.com/willmarple/harness-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
