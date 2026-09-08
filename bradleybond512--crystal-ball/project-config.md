---
trigger: always_on
description: Crystal Ball is a real-time OSINT intelligence dashboard built with **Vanilla TypeScript** (no UI framework). It aggregates 30+ external data sources — geopolitics, military activity, financial markets, cyber threats, climate events — rendered on an interactive 3D globe via MapLibre GL and deck.gl, with a grid of specialised data panels.
---

# GitHub Copilot Instructions — Crystal Ball

## Project Overview

Crystal Ball is a real-time OSINT intelligence dashboard built with **Vanilla TypeScript** (no UI framework). It aggregates 30+ external data sources — geopolitics, military activity, financial markets, cyber threats, climate events — rendered on an interactive 3D globe via MapLibre GL and deck.gl, with a grid of specialised data panels.

Deployment: static SPA on Vercel CDN + 60+ Vercel Edge Functions + optional Tauri desktop shell.

---

## Repository Layout

```
src/components/ UI components — Panel subclasses, map, modals (~400 panels)
src/services/ Data fetching modules — sebuf clients, AI, signal analysis
src/config/ Static data and variant configs (feeds, geo, military, pipelines)
src/generated/ Auto-generated sebuf client + server stubs (do NOT edit by hand)
src/types/ TypeScript type definitions
src/locales/ i18n JSON files (14 languages)
src/workers/ Web Workers for ML/signal analysis
server/ Sebuf handler implementations for 17 domain services
api/ Vercel Edge Functions (sebuf gateway + legacy endpoints)
proto/ Protobuf service and message definitions
data/ Static JSON datasets
src-tauri/ Tauri v2 Rust app + Node.js sidecar (desktop)
e2e/ Playwright end-to-end tests
scripts/ Build and packaging scripts
docs/ Documentation + generated OpenAPI specs
```

---

## Variant System

Three app variants share all source code but differ in default panels, map layers, and RSS feeds:

| Variant | Command | Audience |
|-----------|----------------------|------------------------------------------------|
| `full` | `npm run dev` | Geopolitics, military, conflicts, disaster |
| `tech` | `npm run dev:tech` | Startups, AI/ML, cloud, cybersecurity |
| `finance` | `npm run dev:finance`| Markets, trading, central banks, commodities |

Variant configs live in `src/config/variants/`.

---

## Key Technologies

- **TypeScript** — all code (frontend, edge functions, handlers). Avoid `any`; use `unknown` + type guards.
- **Vite** — build tool / dev server
- **Sebuf** — proto-first HTTP RPC framework. Service definitions in `proto/`, generated stubs in `src/generated/`, handlers in `server/crystalball/{domain}/v1/`
- **Protobuf / Buf** — 17 domain services
- **MapLibre GL** — base map (tiles, globe mode, camera)
- **deck.gl** — WebGL overlay layers (scatterplot, geojson, arcs, heatmaps)
- **d3** — charts, sparklines, data visualization
- **Vercel Edge Functions** — serverless API gateway
- **Tauri v2** — desktop app (macOS, Windows, Linux)
- **Playwright** — E2E and visual regression testing
- **DOMPurify** — HTML sanitization before any DOM insertion

---

## Build & Test Commands

```bash
npm run dev # Start dev server (full variant, port 3000)
npm run dev:tech # Tech variant
npm run dev:finance # Finance variant
npm run typecheck # TypeScript type check (no emit)
npm run typecheck:all # Typecheck frontend + API tsconfigs
npm run test:sidecar # Node native test runner — sidecar + API unit tests
npm run test:data # Data integrity tests
npm run test:e2e # Playwright E2E tests
npm run build # Production build (full variant)
npm run build:desktop # Desktop production build
npm run lockfile:check # Verify package-lock.json integrity
npm run lint:md # Markdown linting
```

Run **`npm run typecheck`** and **`npm run test:sidecar`** before every PR.

## Merge And Main-To-Mac Delivery

- Agent branches such as `copilot/*`, `claude/*`, and `codex/*` must go through PRs and GitHub auto-merge. Do not use direct PR merges to bypass required checks.
- `main` is the only merge target.
- Official desktop releases remain tag-driven.
- Bradley's local Mac install path is synchronized from `main` by `npm run main-sync:setup`, which installs a LaunchAgent that polls `macos/main` from a dedicated clean clone at `~/.crystalball-main-sync/repo`.
- That sync path must verify GitHub required checks for `main`, then run `npm run lockfile:check`, `npm ci`, `npm run version:check`, `npm run typecheck:all`, `npm run build`, `npm run desktop:build:app:full`, and install via `node scripts/install-built-app.mjs --relaunch`.
- Update `AGENTS.md`, `CLAUDE.md`, and these instructions together whenever the sync contract changes.
- Do not add `self-hosted` jobs to PR-triggered workflows in this public repository.

---

## Release and Main Sync

- Agent branches (`claude/*`, `codex/*`, `copilot/*`) must go through PRs and GitHub auto-merge after required checks pass.
- Keep local delivery aligned with the same guarded path by installing and maintaining the LaunchAgent with `npm run main-sync:setup`.
- Trigger a one-off local sync with `npm run main-sync:run`.

---

## Coding Conventions

### TypeScript

- `const` by default, `let` only when reassignment is needed
- Prefer functional patterns (`map`, `filter`, `reduce`) over imperative loops
- Export interfaces/types for all public APIs
- JSDoc on all exported functions and non-obvious logic
- No `any` — use proper types or `unknown` with type guards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradleybond512/crystal-ball](https://github.com/bradleybond512/crystal-ball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
