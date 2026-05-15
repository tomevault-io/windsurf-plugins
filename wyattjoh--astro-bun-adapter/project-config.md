---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Astro adapter for Bun (`@wyattjoh/astro-bun-adapter`). Enables Astro SSR sites to run on `Bun.serve` with optimized static file serving via a pre-computed manifest and ISR (Incremental Static Regeneration) support.

## Commands

- **Build**: `bun run build` — runs `bunup` to bundle `src/index.ts`, `src/server.ts`, and `src/cache.ts` (target: bun, format: esm, with declarations, sourcemaps, all packages external)
- **Test**: `bun run test` — runs Bun's built-in test runner concurrently on `src/`
- **Single test**: `bun test src/path/to/file.test.ts`
- **Typecheck**: `bun run typecheck` — runs `tsc --noEmit`
- **Lint**: `bun run lint` — runs Biome (`biome check .`)
- **Lint fix**: `bun run lint:fix` — runs Biome with auto-fix
- **Format**: `bun run format` — runs Biome formatter

Pre-commit hook (via Husky) runs lint, test, and typecheck automatically.

## Architecture

Source files in `src/`:

- **`index.ts`** — The Astro integration entry point. Exports `bun()` which hooks into Astro's build lifecycle: configures build settings at `astro:config:setup` (including a Vite virtual module plugin for config injection and `entrypointResolution: "auto"`), registers the adapter with `staticHeaders` and `middlewareMode` adapter features at `astro:config:done`, generates the static manifest and build ID at `astro:build:done`.
- **`vite-plugin-config.ts`** — Vite 7 virtual module plugin factory. Exposes adapter config (ISR options, static cache control, etc.) to the server entrypoint at runtime via a virtual import (`virtual:@wyattjoh/astro-bun-adapter/config`), avoiding the need to serialize args into the entry file.
- **`virtual.d.ts`** — TypeScript declarations for the `virtual:@wyattjoh/astro-bun-adapter/config` virtual module.
- **`server.ts`** — The runtime server entrypoint. Imports adapter config from the virtual module (`virtual:@wyattjoh/astro-bun-adapter/config`) and creates the Astro app via `createApp()` from `astro/app/entrypoint`. Boots `Bun.serve` at the top level (no `start()` function). Looks up requests against the static manifest for direct file serving with ETag/304 support and `x-astro-cache: STATIC` headers, falls back to SSR. Wires `clientAddress` from `Bun.serve`'s `requestIP()`. Bypasses ISR for server island requests. Integrates ISR when enabled. Normalizes image endpoint query params into deterministic cache keys. Registers `SIGTERM`/`SIGINT` handlers for graceful shutdown (flushes ISR cache to disk).
- **`isr/handler.ts`** — ISR request handler. Wraps SSR origin with cache lookup/store logic. Respects `s-maxage` and `stale-while-revalidate` from `Cache-Control` headers. Deduplicates concurrent requests for the same path. Overrides Astro's image endpoint `Cache-Control` to add `s-maxage` so image responses are ISR-cacheable. Tags responses with `x-astro-cache` header (`HIT`/`STALE`/`MISS`/`BYPASS`).
- **`isr/cache.ts`** — `PersistentLRUCache`: two-tier byte-limited LRU cache. L1 is an in-memory doubly-linked list; L2 is per-entry CBOR files on disk (`{cacheDir}/{buildId}/entries/{hash}.cbor`). Evicted entries remain on disk and reload on demand. Debounced index writes, concurrent disk-read deduplication, optional memory pre-fill on startup, and automatic vacuuming of old build cache directories.
- **`manifest.ts`** — Build-time utility. Walks `dist/client/`, hashes files (SHA-256, truncated), and writes `dist/server/.astro-bun-adapter/static-manifest.json`. Generates clean URL route aliases for pre-rendered HTML pages (e.g. `/about` → `/about/index.html`). Merges `staticHeaders` route-level headers into manifest entries. Uses `node:fs/promises` and `node:crypto` because Astro build hooks run under Node, not Bun.
- **`cache.ts`** — Public API module for on-demand ISR cache expiration. Exports `unstable_expirePath(pathname)` (deletes a cached entry so it is lazily re-rendered on the next request) and `unstable_expireAll()` (clears all cached entries). Also exports `registerCache()` (internal, called by `server.ts` at startup). Uses `Symbol.for()` on `globalThis` to share the cache reference across module boundaries regardless of bundling.
- **`types.ts`** — Shared types (`AdapterOptions`, `ISROptions`, `ISRCache`, `ISRHandler`, `ManifestEntry`, `StaticManifest`, `ISRCacheEntry`).

## Key Design Decisions

- Build hooks run under **Node**, so `manifest.ts` must use `node:` APIs, not Bun APIs.
- The runtime server uses the **Web-standard `App`** (not `NodeApp`) since Bun natively supports the Fetch API.
- Adapter config is injected into the server entrypoint via a Vite virtual module (`virtual:@wyattjoh/astro-bun-adapter/config`) instead of the deprecated `args` mechanism. The config is JSON-serialized into `export const` statements at build time. At runtime, `app.manifest` provides `buildClientDir`, `buildServerDir`, and `base`.
- `/_astro/*` paths get immutable 1-year cache headers; everything else defaults to 24-hour must-revalidate (configurable via the `staticCacheControl` adapter option). Route-level `staticHeaders` take precedence over `staticCacheControl`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyattjoh/astro-bun-adapter](https://github.com/wyattjoh/astro-bun-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
