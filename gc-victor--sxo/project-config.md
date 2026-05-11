---
trigger: always_on
description: - `pnpm test` - run all tests
---

# AGENTS.md

## Quick Start (TL;DR)

**Commands:**

- `pnpm test` - run all tests
- `pnpm run test:file src/js/path/to/file.test.js` - run single test file (dedicated script with `NODE_ENV=test`)
- `npm run check` - lint with Biome
- `npm run format` - format code
- `./bin/sxo.js create|add|dev|build|start|clean|generate`

**Architecture:** ESM-only Node 20+ SSR framework. Dual esbuild outputs ([`dist/client`](dist/client) public, [`dist/server`](dist/server) SSR bundles). Route discovery from `pages/` dir, manifest at [`dist/server/routes.json`](dist/server/routes.json). JSX via Rust/WASM transformer. Middleware in [`src/middleware.js`](src/middleware.js).

**Style:** Small modules, ESM imports, JSDoc on exported utilities. No React patterns. Pages return full HTML docs (`<html>` + `<head>`). See [`.rules/jsx-standards.md`](.rules/jsx-standards.md) for examples, [`.rules/jsdoc.md`](.rules/jsdoc.md) for docs, [`.rules/testing.instructions.md`](.rules/testing.instructions.md) for tests.

**Constraints:** Only modify [`src/js/**`](src/js) unless directed. Never edit [`jsx-transformer/jsx_transformer.js`](jsx-transformer/jsx_transformer.js), [`dist/**`](dist), or generated artifacts. No mega-refactors (>300 LOC / >3 files) without confirmation. Preserve `AIDEV-*` anchors.

**Key Files:** [`src/js/esbuild/`](src/js/esbuild) (build pipeline), [`src/js/server/`](src/js/server) (dev/prod servers), [`src/js/config.js`](src/js/config.js) (resolution), [`src/js/cli/`](src/js/cli) (commands). Read full sections below for details.

**Loaders:** Custom esbuild server loaders can be configured via CLI flags (`--loaders ".svg=file"`), env (`LOADERS='{"svg":"file"}'`), or config file (`{"loaders":{".svg":"file"}}`). Format supports both `.svg` and `svg` notation (dot auto-added). Repeatable flags merge together; comma-separated values supported. Only propagated to dev/build commands. Follow standard config precedence: flags > file > env > defaults.

---

## Special Error Pages (404/500)

This project supports root-level 404 and 500 pages that render as full HTML documents to replace simple text fallbacks.

- Location and filenames:
  - PAGES_DIR/404.(tsx|jsx|ts|js)
  - PAGES_DIR/500.(tsx|jsx|ts|js)
- Exports and semantics:
  - Accepts default export or named export `jsx` (server uses `module.default || module.jsx`).
  - Pages must return a full HTML document and include their own `<head>` (head injection is removed).
  - These special pages are not routable and are not added to the manifest as public routes.

- Build and manifest:
  - The server build includes 404/500 SSR modules so they can be imported at runtime.
  - No manifest schema changes; these pages do not receive route paths or asset mappings at this time.
  - Static generation does not generate 404/500 pages.
- Response semantics:
  - HEAD requests: responses include headers only (no body) for 404/500 (custom or fallback).
  - Cache-Control: 404 → `public, max-age=0, must-revalidate`; 500 → `no-store`.

## Info

- **Project**: sxo
- **Last Update**: 2025-11-11
- **Rules**: [`.rules/`](.rules)

## Purpose

Authoritative onboarding & guard-rails for AI + human contributors. Read fully before non-trivial changes.

---

## 0. Project Overview

- Manifest path: [`dist/server/routes.json`](dist/server/routes.json) (NOT `dist/routes.json`).
- Dual build outputs: [`dist/client`](dist/client) (public) / [`dist/server`](dist/server) (private).
- Page module acceptance: **default export OR named `jsx`** (server picks `module.default || module.jsx`).
- Middleware system: `SRC_DIR/middleware.js` (hot-replace in dev).
- Head injection removed: pages return full `<html>` and manage their own `<head>` contents directly.
- Static asset server supports: hashed caching, ETag, precompressed variants, range requests (uncompressed only).
- Hot reload: SSE endpoint `/hot-replace?href=<path>` with partial body replacement.
- Public asset base path configurable via `--public-path`, `PUBLIC_PATH`, or config; empty string "" preserved; consumed by esbuild `publicPath`; normalized at runtime for injection (empty string preserved → no leading slash; non‑empty ensures trailing slash).
- Per‑route client entry subdirectory configurable via `clientDir` (config), `CLIENT_DIR` (env), or `--client-dir` (flag). Default: "client".
- Custom esbuild server loaders configurable via `--loaders` (flag), `LOADERS` (env), or config; follows precedence: flags > file > env > defaults; only propagated to dev/build commands.
- Static generation support: `sxo generate` pre-renders non-dynamic routes, writes HTML into [`dist/client`](dist/client), and marks routes with `generated: true` in the manifest.
- Prod server respects `generated` flag: if `generated: true`, serves built HTML as-is (skips SSR) with `Cache-Control: public, max-age=300`; otherwise SSR per request with `Cache-Control: public, max-age=0, must-revalidate`.
- Prod timeouts: `REQUEST_TIMEOUT_MS` (default 120000) sets `server.requestTimeout`; `HEADER_TIMEOUT_MS` (if set to a non-negative integer) overrides `server.headersTimeout`.

---

## 1. Golden Rules

- Do not invent architecture—ask if ambiguous.
- Only modify code under [`src/js/**`](src/js) unless explicitly directed.
- Preserve existing `AIDEV-*` anchors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gc-victor/sxo](https://github.com/gc-victor/sxo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
