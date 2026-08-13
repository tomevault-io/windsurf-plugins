---
trigger: always_on
description: > **Note:** Piwi Dashboard is **not affiliated with, endorsed by, or connected to Microsoft Corporation** in any way.
---

# Piwi Dashboard — Agent Instructions

> **Note:** Piwi Dashboard is **not affiliated with, endorsed by, or connected to Microsoft Corporation** in any way.
> The name "Piwi" was chosen as a playful, unrelated name. The project was originally called "Playwright Dashboard" and
> was renamed to avoid confusion with Microsoft's Playwright testing framework.

Root guide for any AI agent (Claude Code, opencode, Copilot, Cursor, …). It covers the whole monorepo: what lives where,
how to run and verify things, and the conventions that apply everywhere.

**Area guides — read the one covering the directory you are editing, in addition to this file:**

| Editing… | Read first |
|---|---|
| `apps/application/` — the Nuxt dashboard (app, server, demo, MCP) | [`apps/application/AGENTS.md`](apps/application/AGENTS.md) |
| `packages/reporter/` — the Playwright reporter package | [`packages/reporter/AGENTS.md`](packages/reporter/AGENTS.md) |
| `apps/desktop/` — the Tauri desktop shell | [`apps/desktop/AGENTS.md`](apps/desktop/AGENTS.md) |
| `apps/extension/` — the browser extension (Manifest V3) | [`apps/extension/AGENTS.md`](apps/extension/AGENTS.md) |
| `apps/docs/` — the VitePress documentation site | [`apps/docs/AGENTS.md`](apps/docs/AGENTS.md) |

Reference material worth opening when you need the map rather than the rules:
[`apps/application/ARCHITECTURE.md`](apps/application/ARCHITECTURE.md) (dashboard) and
[`packages/reporter/ARCHITECTURE.md`](packages/reporter/ARCHITECTURE.md) (reporter).

## Project overview

A test results dashboard for **Playwright**, built with **Nuxt 4** and **Nuxt UI**. It ingests test results from a custom
reporter, stores them (SQLite or PostgreSQL), and adds analysis on top: flaky detection, failure clustering, AI diagnosis,
locator healing, notifications and an MCP server for AI agents. Self-hosted only — no SaaS, no other test frameworks
(see [`ROADMAP.md`](ROADMAP.md) for direction and non-goals).

## Repository layout

```
apps/                      Deployable surfaces — the things you run, install, download or read.
  application/             Nuxt 4 dashboard — app (UI), server (API), demo SPA, MCP server
  application/shared/      Types, constants & pure utilities shared app-wide (import via `#shared/...`)
  desktop/                 Tauri desktop shell that bundles and runs the same server locally
  extension/               Piwi Picker — browser extension (Manifest V3), standalone, no server dependency
  docs/                    VitePress documentation site, published to GitHub Pages
packages/                  Packages consumed by name (`@piwitests/*`) — published to npm or imported by a workspace.
  core/                    @piwitests/core — private, zero-dependency logic shared by app AND reporter
  picker-dom/              @piwitests/picker-dom — shared DOM picker overlay (reporter, dashboard, extension)
  reporter/                @piwitests/reporter — the Playwright reporter (TypeScript → bundled via tsup)
  server/                  @piwitests/server — published npm run-option (`npx @piwitests/server`)
integrations/              Framework-specific instrumentation adapters.
  nitro/                   @piwitests/instrumentation-nitro — backend-log instrumentation for Nitro apps
  aspnetcore/              PiwiTests.Instrumentation.AspNetCore — the same for ASP.NET Core (NuGet)
examples/                  Standalone usage examples (Playwright fixtures)
shared/                    Base oxlint/oxfmt configs extended by every workspace
plans/                     Local working docs — gitignored, never committed
```

**Where a new workspace goes** — the split is deliberate, keep it consistent:

- `apps/*` — a deployable surface: something you run, install, download or read (the dashboard, the desktop app, the
  extension, the docs site).
- `packages/*` — a package consumed *by name* (`@piwitests/*`): either published to npm (`reporter`, `server`) or
  imported by another workspace (`core`, `picker-dom`).
- `integrations/*` — a framework-specific instrumentation adapter, one directory per framework.

Every JS workspace is listed in the root [`package.json`](package.json) `workspaces` array. Three directories carry a
`package.json` but are **intentionally not workspaces**: `apps/desktop` and `apps/docs` install and build on their own
toolchains (Tauri, VitePress) rather than through the root install, and `examples/playwright-fixtures` must resolve
`@piwitests/*` from the published npm registry — release-please bumps its pinned versions — so making it a workspace
would symlink the local copies and defeat the example.

`plans/` holds two tracked-by-hand files: `plans/roadmap.md` (working priorities) and `plans/exploration-findings.md`
(a log of bugs, tech debt and inconsistencies found while exploring). Both are local-only. Public direction lives in the
committed [`ROADMAP.md`](ROADMAP.md).

## Quick start

Prerequisites: **Node.js 24+**, npm, Git. Commands run from `apps/application/` unless noted.

```bash
cd apps/application
npm install
npm run app:dev      # http://localhost:3000
```

The SQLite database and `.data/` storage are created automatically on the first API call — no configuration needed.

## Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PiwiTests/platform](https://github.com/PiwiTests/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
