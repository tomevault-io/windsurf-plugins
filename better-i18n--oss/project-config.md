---
trigger: always_on
description: Open-source SDK ecosystem for the **Better i18n** localization platform. These packages are used by **customers** in their production applications — code quality, backwards compatibility, and reliability are critical.
---

# Better i18n OSS — Claude Code Context

## What This Is

Open-source SDK ecosystem for the **Better i18n** localization platform. These packages are used by **customers** in their production applications — code quality, backwards compatibility, and reliability are critical.

**Platform repo:** `better-i18n` (private) — API, dashboard, sync workers, CDN worker
**This repo:** `better-i18n-oss` (public) — SDKs, CLI, MCP servers that customers install

## AI Assistant Guidelines

- **Package manager:** Bun — use `bun install`, `bun test`, `bun run build`
- **Test runner:** Vitest (root `vitest.config.ts`), except `packages/cli` which uses bun:test
- **Build:** `tsc` per-package (no bundler)
- **Language:** TypeScript 5.9, ESNext modules, `"moduleResolution": "Bundler"`
- **NEVER introduce breaking changes** without explicit approval — customers depend on these packages
- **ALWAYS run tests** after modifying any package: `bun test packages/{name}`
- **ALWAYS use changesets** for version bumps: `bunx changeset`
- **ALWAYS update Linear tickets after commit** — If the task has a Linear issue (BETTER-xxx), update its status to "Done" using `mcp__linear-server__save_issue` after committing. Reference the ticket ID in the commit message (e.g., `feat: add feature [BETTER-105]`). Tickets must not be left open after work is completed.

## Monorepo Structure

```
better-i18n-oss/
├── packages/
│   ├── core/          # Foundation — CDN fetch, TtlCache, locale utils (ALL packages depend on this)
│   ├── use-intl/      # React + TanStack Router adapter (wraps use-intl)
│   ├── next/          # Next.js + next-intl adapter with ISR
│   ├── expo/          # React Native / Expo adapter (i18next)
│   ├── server/        # Server-side: Hono middleware + Node.js adapter
│   ├── remix/         # Remix / Shopify Hydrogen adapter
│   ├── cli/           # CLI: scan, sync, check, doctor commands
│   ├── sdk/           # Headless CMS content client (Supabase-style API)
│   ├── mcp/           # MCP server for translation management (AI agents)
│   ├── mcp-content/   # MCP server for content management
│   ├── mcp-types/     # Shared types — synced from internal repo, DO NOT edit manually
│   ├── schemas/       # Shared Zod schemas
│   └── flutter/       # Flutter/Dart SDK
├── apps/
│   ├── landing/       # Marketing site (TanStack Start, dogfoods use-intl + sdk)
│   ├── docs/          # VitePress documentation
│   ├── hono-example/  # Example Hono server
│   └── hydrogen-demo/ # Shopify Hydrogen example
```

## Package Dependency Graph

```
@better-i18n/core          ← Foundation (no deps)
       ↓
@better-i18n/use-intl      (core + use-intl + @tanstack/react-router)
@better-i18n/next          (core + next-intl + next)
@better-i18n/expo          (core + i18next)
@better-i18n/server        (core + use-intl/core + hono)
@better-i18n/remix         (core)

@better-i18n/mcp-types     ← private, synced from internal
       ↓
@better-i18n/mcp           (mcp-types + @trpc/client + @modelcontextprotocol/sdk)
@better-i18n/mcp-content   (mcp-types + @modelcontextprotocol/sdk)

@better-i18n/sdk           (standalone)
@better-i18n/cli           (standalone)
```

**CRITICAL:** Changes to `@better-i18n/core` affect ALL downstream packages. Test thoroughly.

## CDN Architecture (Platform Side)

Understanding the CDN is essential for working on SDKs — this is what SDKs talk to.

### CDN URL Pattern

```
https://cdn.better-i18n.com/{org}/{project}/manifest.json
https://cdn.better-i18n.com/{org}/{project}/{locale}/translations.json
```

### How the CDN Worker Serves Translations

```
SDK fetch → CDN Worker (CF Worker on cdn.better-i18n.com)
                ↓
          CF Cache API (L1) — 60s translations, 300s manifest
                ↓ (miss)
          R2 Bucket (L2) — source of truth, written by sync-worker
                ↓ (timeout/error)
          Stale Cache (L3) — resilience fallback
                ↓ (nothing)
          {} or { fallback: true } — always HTTP 200, never 5xx
```

**Key behaviors:**
- CDN **always returns HTTP 200** — even on errors, SDKs get `{}` or `{ fallback: true }`
- Response headers: `Cache-Control: public, max-age=60, s-maxage=60` (translations), `max-age=300, s-maxage=600` (manifest)
- Analytics Engine writes per-request: language, namespace, cache status, bytes, latency

### Publish → Purge → Fresh Data Flow

```
User publishes translations
  → Sync Worker uploads JSON to R2
  → Sync Worker fires purge request to CDN Worker (fire-and-forget)
  → CDN Worker: R2.list() → caches.default.delete() per file + manifest
  → Next SDK request: Cache MISS → R2 read → fresh data
```

**Purge is non-critical** — if it fails, `max-age=60` ensures data refreshes within 60s anyway.

### Cache Layers Across Platforms (After Publish + Purge)

| Platform | Max Stale Duration | Cache Source |
|----------|-------------------|-------------|
| Browser SPA | ~60s | CDN `max-age=60` + SDK TtlCache (60s) |
| Next.js (ISR) | ~30s | ISR `revalidate: 30` for messages |
| TanStack Start (CF Worker SSR) | ~60s | SDK TtlCache (60s) — `no-store` bypasses CF subrequest cache |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [better-i18n/oss](https://github.com/better-i18n/oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
