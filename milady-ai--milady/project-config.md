---
trigger: always_on
description: Milady is a local-first AI assistant built on [elizaOS](https://github.com/elizaOS). It wraps the elizaOS runtime with a CLI, desktop app (Electrobun), web dashboard, and platform connectors (Telegram, Discord, etc.).
---

# Milady — Agent Conventions

## What This Is

Milady is a local-first AI assistant built on [elizaOS](https://github.com/elizaOS). It wraps the elizaOS runtime with a CLI, desktop app (Electrobun), web dashboard, and platform connectors (Telegram, Discord, etc.).

### elizaOS naming (agents & editors)

Write the framework name as **elizaOS** in prose, comments, user-facing strings, and documentation — not `ElizaOS`. The npm scope remains **`@elizaos/*`** (lowercase). Say **Eliza agents** when you mean agents in plain language (not **elizaOS agents**). The **Eliza Classic** plugin name is an exception (**Eliza** = the 1966 chatbot), not “elizaOS Classic”. Cursor picks this up via `.cursor/rules/elizaos-branding.mdc`.

## Quick Start (Dev)

```bash
bun install          # runs postinstall hooks automatically
bun run dev          # API on :31337, UI on :2138 with hot reload (defaults; busy ports → next free + env sync)
bun run dev:desktop  # Electrobun; skips vite build when apps/app/dist is up to date
bun run dev:desktop:watch  # Vite **dev** server + Electrobun `MILADY_RENDERER_URL` (HMR). Orchestrator pre-picks free API/UI loopback ports when defaults are in use so proxy + env match. Rollup watch: also set MILADY_DESKTOP_VITE_BUILD_WATCH=1

Desktop dev observability (agents cannot see the native window; Cursor does not auto-poll localhost): `GET /api/dev/stack` on the API; `bun run desktop:stack-status -- --json`; default-on aggregated log (`.milady/desktop-dev-console.log`) + `GET /api/dev/console-log` (loopback tail); default-on screenshot proxy `GET /api/dev/cursor-screenshot` (loopback, full-screen OS capture). Opt-out: `MILADY_DESKTOP_SCREENSHOT_SERVER=0`, `MILADY_DESKTOP_DEV_LOG=0`. See `docs/apps/desktop-local-development.md` and `.cursor/rules/milady-desktop-dev-observability.mdc`.
```

Desktop dev rationale (signals, Quit, `detached` children): `docs/apps/desktop-local-development.md`.

Optional — link a local elizaOS source checkout for live package development:
```bash
bun run setup:upstreams   # initializes repo-local ./eliza and links local @elizaos/* packages
```

## Build & Test

```bash
bun run build        # tsdown + vite
bun run verify       # typecheck + lint (`bun run check` aliases this)
bun run test         # parallel test suite
bun run test:e2e     # end-to-end tests
bun run db:check     # database security + readonly tests
```

## Project Layout

```
packages/
  app-core/             Main application package (source of truth for runtime)
    src/
      entry.ts          CLI bootstrap (env, log level)
      cli/              Commander CLI (milady command)
      runtime/
        eliza.ts        Agent loader — sets NODE_PATH, loads plugins dynamically
        dev-server.ts   Dev mode entry point (started by dev-ui.mjs)
      api/              Dashboard API (port 31337 in dev, 2138 in prod)
      config/           Plugin auto-enable, config schemas
      connectors/       Connector integration code
      services/         Business logic
  agent/                Upstream elizaOS agent (core plugins, auto-enable maps)
  plugin-wechat/        WeChat connector plugin (@elizaos/plugin-wechat)
  ui/                   Shared UI component library
  shared/               Shared utilities
apps/
  app/                  Main web + desktop UI (Vite + React)
    electrobun/         Electrobun desktop shell
  homepage/             Marketing site
scripts/
  dev-ui.mjs            Dev orchestrator (API + Vite)
  eliza/packages/app-core/scripts/run-node.mjs   CLI runner (spawns entry.js with NODE_PATH)
  run-repo-setup.mjs    Postinstall sequencer
  setup-upstreams.mjs   Initialize repo-local upstreams and link @elizaos packages
  patch-deps.mjs        Post-install patches for broken upstream exports
```

## Default Agent Knowledge

Treat bundled skills from `@elizaos/skills` as the default knowledge base for code agents working in this repo. Repo setup mirrors those defaults into `skills/.defaults/` so task agents like Claude and Codex can open them directly from the workspace. The canonical repo-local entry points are:

- `skills/.defaults/eliza-app-development/SKILL.md` — this repo as an elizaOS app (Milady is this checkout’s product name), layout, and how local, remote, and cloud paths fit together
- `skills/.defaults/elizaos/SKILL.md` — elizaOS runtime concepts, plugin abstractions, and extension points
- `skills/.defaults/eliza-cloud/SKILL.md` — Eliza Cloud as a managed backend, app platform, deployment target, and monetization surface

The source of truth remains under `eliza/packages/skills/skills/`. `scripts/sync-workspace-default-skills.mjs` refreshes the repo-local mirrors during repo setup.

`scripts/ensure-skills.mjs` seeds bundled skills from `@elizaos/skills` into the managed skills store on first run.
Separately, `packages/agent/src/runtime/default-knowledge.ts` seeds bundled runtime knowledge items for Milady itself, including the baseline Eliza Cloud app/backend guidance.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milady-ai/milady](https://github.com/milady-ai/milady) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
