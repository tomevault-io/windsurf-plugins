---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Workbench (sf-toolkit-web) — a Salesforce administration toolkit shipped as a Chrome extension, desktop Electron app, and hosted web app. Provides an overlay panel, a Monaco-based VS Code editor, metadata explorer, SOQL editor, and an AI agent that can control the browser. Node **22.14** is required (see `package.json` engines).

## Common Commands

```sh
# Install (runs LWR HMR patch + installs desktop subpackage)
npm install

# Web app dev / prod
npm run start:dev:web              # via start:dev:server
npm run start:dev:server           # NODE_ENV=development, loads .env.dev
npm run start:prod:server          # NODE_ENV=production, loads .env.prod

# Chrome extension
npm run start:dev:extension        # watch + serve dist/extension
npm run build:prod:extension       # full prod build (main + sandbox + workers)
npm run build:extension:main       # main-only (faster)

# Desktop (Electron) — lives under packages/desktop
npm run start:dev:desktop          # against an already-running web server
npm run start:dev:desktop:all      # watch extension + start desktop together
npm run build:desktop
npm run desktop:open               # launcher CLI

# Shared TS + generated manifests (runs before most builds)
npm run build:shared               # generate_manifest_skill + generate_application_manifest + tsc on packages/lwc/shared

# Workers
npm run watch:workers
npm run build:workers

# Vendor bundles (required once after clone, and after vendor updates)
npm run build:vendor:just-bash     # built into vendor-bundles/just-bash, copied to assets/extension/libs

# Landing site + docs (apps/ui, apps/docs — React + Vite / Docusaurus)
npm run site:dev                   # runs apps/ui + apps/docs in parallel
npm run site:build

# Local MCP test server (for AI → MCP → Config validation)
npm run start:test:mcp             # http://localhost:3999/mcp
npm run test:mcp                   # automated smoke test

# Live LLM provider harness (internal gateway — requires WORKBENCH_GATEWAY_KEY)
# Dumps JSONL chunks under tools/llm-provider-harness/out/ for review.
WORKBENCH_GATEWAY_KEY=sk-... npm run test:provider:internal:streaming
WORKBENCH_GATEWAY_KEY=sk-... npm run test:provider:internal:non-streaming
WORKBENCH_GATEWAY_KEY=sk-... npm run test:provider:internal  # both, in sequence

# Quality
npm run lint                       # eslint + prettier --check
npm run format                     # prettier --write
npm run test                       # node --test on packages/lwc/main/**/__tests__/*.test.js
npm run validate                   # lint + test + build:all

# Single test file
node --experimental-strip-types --test packages/lwc/main/path/to/__tests__/foo.test.js
```

If HMR misbehaves during `start:dev:web`: `rm -rf __lwr_cache__` and restart. The LWR namespace patch (`tools/scripts/patch_lwr_hmr_namespace.mjs`) is auto-applied by `postinstall`.

## Architecture

### Monorepo layout

```
apps/ui                       Landing/welcome site (React + Vite)
apps/docs                     Docusaurus docs
packages/lwc/main             Host LWC shell + core services + host-api
packages/lwc/applications     Pluggable feature apps (SOQL, metadata, api, object, …)
packages/lwc/extension        Chrome-extension-specific LWC surfaces (overlay, panels, feature)
packages/lwc/shared/modules   Cross-target pure utilities (shared/utils, shared/logger, shared/llm, …)
packages/server               LWR dev/prod server, OAuth, content, layouts, hooks
packages/extension/src        Chrome extension entry points + manifest.template.json
packages/desktop              Electron app
packages/vscode               VS Code webview (Monaco, React — NOT LWC; out of host-api scope)
packages/workers/src          Rollup-built web workers
vendor-bundles/just-bash      Pre-built vendor browser bundles
tools/build                   Rollup configs (extension, workers)
tools/scripts                 Generators + LWR HMR patch + asset sync scripts
tools/mcp                     Local MCP test server + sample config
tools/llm-provider-harness    Live LLM provider harness (internal gateway) — streaming + non-streaming
assets                        Design-system assets, skills manifest, screenshots
```

### Core architectural split: host ↔ apps

The repo is mid-refactor from monolith into a **core host + pluggable apps** model (inspired by VS Code extensions). See `packages/lwc/main/host-api/README.md` for the contract — it is the single most important doc for understanding boundaries.

- **Host** (`packages/lwc/main/*`): shell/chrome, routing, Redux store, connector, design system, agent runtime, `host-api/`.
- **Apps** (`packages/lwc/applications/<name>/`): self-contained launchable features. Each has its own `package.json`, may ship a `<name>.manifest.json`, and registers through the application registry.
- **`host-api/`** = the stable contract apps are allowed to import. Apps must **not** reach into `core/*` directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grebmann1/workbench](https://github.com/grebmann1/workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
