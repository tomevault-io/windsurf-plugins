---
trigger: always_on
description: **Generated:** 2026-08-21
---

# HOLY GRAIL — Project Knowledge Base

**Generated:** 2026-08-21
**Branch:** grail

## OVERVIEW

Curated directory of developer tools, AI platforms, browser extensions, and learning resources. Vue 3 SPA with Vite 8, TypeScript 6, Tailwind CSS 4, Pinia 3, Vue Router 5, Supabase, PostHog. Hosted on Vercel. Content-driven via `meta.yaml` files.

Monorepo: **Bun workspaces** (`packages/*`) with a single lockfile, orchestrated by **Turborepo** (`turbo.json`). The web app is the workspace root package sibling — `packages/web` is the Vercel deployment root (vite root, `api/` functions, `public/` submodule).

## STRUCTURE

```
├── packages/
│   ├── web/            # Vue 3 SPA — deployment root for Vercel
│   │   ├── src/        #   components, pages, stores, composables, content
│   │   ├── public/     #   static assets + runtime JSON; previews submodule mounts here
│   │   ├── api/        #   Vercel serverless functions (/api/mcp → /mcp via vercel.json)
│   │   ├── scripts/    #   build generators, preview pipeline, enrichment (Bun + Python)
│   │   ├── tests/      #   Vitest tests
│   │   ├── index.html, vite.config.ts, tsconfig*.json, env.d.ts, vercel.json
│   │   └── package.json #  holy-grail-web (private)
│   ├── mcp/            # holy-grail-mcp — publishable npm package (MCP server)
│   ├── cli/            # grail-cli — publishable npm package (Rust+TS CLI)
│   ├── supabase/       # Edge Functions (Deno) + DB migrations — infra, no turbo tasks
│   └── docs/           # Documentation + AGENTS.md
├── turbo.json          # Task graph: build / type-check / dev (cargo + Deno NOT cached)
├── package.json        # Workspace root: workspaces + turbo + repo-wide lint/format scripts
├── .gitmodules         # Registers packages/web/public/previews → Drakaniia/holy-grail-assets
├── .githooks/          # Committed pre-commit hook (runs `bun run sync:previews`)
├── .github/            # CI workflows (type-check → lint → build → format:check)
├── .agents/            # OpenCode agent skills
├── .opencode/          # OpenCode runtime config
└── .vibe/              # Vibe config
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add a site | `packages/web/src/content/sites/<category>/<slug>/meta.yaml` | Run generator + preview after (preview lands in the submodule) |
| Add an extension | `packages/web/src/content/extensions/<category>/<slug>/meta.yaml` | Run generator after |
| Add MCP server | `packages/web/src/content/mcp/<slug>/meta.yaml` | Run generator after |
| Modify UI component | `packages/web/src/components/<feature>/` | Feature-grouped |
| Add a page | `packages/web/src/pages/` + `packages/web/src/router/index.ts` | Lazy-loaded |
| Modify store | `packages/web/src/stores/<domain>.ts` | Pinia |
| Add composable | `packages/web/src/composables/use*.ts` | |
| Modify CI | `.github/workflows/` | 4 workflows |
| Modify deploy config | `packages/web/vercel.json` | Vercel SPA (deploys run from `packages/web`) |
| Edit CLI behavior | `packages/cli/src/main.rs` | Rust source |
| Edit MCP server | `packages/mcp/src/` | `bun run build:mcp` (tsc → dist + data snapshot + Vercel function bundle) |
| Add site preview | `bun run generate:previews --slug <slug>` | Puppeteer → WebP into `packages/web/public/previews/` (submodule worktree) |
| Fresh clone setup | `bun run setup` | Init previews submodule + install pre-commit hook |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `src/main.ts` | entry | `packages/web/src/main.ts` | Bootstraps PostHog, theme, Pinia, router |
| `src/App.vue` | component | `packages/web/src/App.vue` | Root layout shell (Navbar, Sidebar, RouterView) |
| `src/router/index.ts` | config | `packages/web/src/router/index.ts` | All routes + auth guard |
| `useSitesStore` | store | `packages/web/src/stores/sites.ts` | Site catalog state |
| `useSkillsStore` | store | `packages/web/src/stores/skills.ts` | Skill catalog state |
| `useExtensionsStore` | store | `packages/web/src/stores/extensions.ts` | Extension catalog state |
| `Site` | type | `packages/web/src/stores/sites.ts` | Core domain model (24 callers) |
| `useSmartSearch` | composable | `packages/web/src/composables/useSmartSearch.ts` | Cross-entity search |
| `generateSitePreviews` | script | `packages/web/scripts/previews/` | Puppeteer screenshot pipeline |
| `createServer` | MCP | `packages/mcp/src/server.ts` | Registers all 10 tools + resources |
| `searchCatalog` | MCP | `packages/mcp/src/search.ts` | Ported SPA scoring (mirrors `useSmartSearch`) |
| `grail` | CLI | `packages/cli/src/main.rs` | Rust skill-management binary |

## CONVENTIONS

- **Bun only.** Never npm/yarn/pnpm. Node 24.x. Single root `bun.lock`; no per-package lockfiles.
- **Turborepo for the task graph**: `turbo run build|type-check|dev` (root `turbo.json`). `cargo` and Deno builds are **not** turbo tasks — `build:cli` shells to cargo directly, `packages/supabase` has no tasks.
- **Vue 3 Composition API + `<script setup lang="ts">`** mandatory.
- **`@/` path alias** → `./src/` (scoped to `packages/web`).
- **Content as YAML**: sites/extensions/mcp defined as `meta.yaml` files, flat-indexed to JSON at build.
- **3 generators run before every dev/build**: sites-index, extensions-index, mcp-index (`packages/web/scripts/build/`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Drakaniia/holy-grail](https://github.com/Drakaniia/holy-grail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
