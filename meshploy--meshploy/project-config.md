---
trigger: always_on
description: Internal Developer Platform. Go Workspaces monorepo + Vite/React frontend deployed via Docker Compose.
---

# Meshploy — Monorepo Rules & Architecture

Internal Developer Platform. Go Workspaces monorepo + Vite/React frontend deployed via Docker Compose.

---

## Repository layout

```
meshploy/
├── apps/
│   ├── api/          # Chi + Huma REST API (Go, OpenAPI 3.1)
│   ├── proxy/        # Edge reverse proxy — "Ask & Resolve" L7 routing
│   ├── cli/          # Static Go binary — node & cluster management CLI
│   ├── web/          # Vite + React 19 + TanStack Router frontend
│   ├── builder/      # Builder image (meshploy-build): git clone + Railpack / Dockerfile
│   └── docs/         # Astro Starlight docs site, generated from the repo's READMEs by sync-docs.mjs
├── packages/
│   ├── db/           # Shared GORM + PostgreSQL models (imported by api and proxy)
│   ├── client/       # Typed Go REST client for the API (imported by cli and mcpserver)
│   ├── mcpserver/    # MCP tool definitions (imported by cli for stdio, api for remote /mcp)
│   ├── proxy/        # The edge proxy's code: route cache, handler, TCP routes (apps/proxy is its one-call main)
│   ├── license/      # Enterprise licence verification: claims, keys, features
│   ├── hostagent/    # Contract between the gateway's host agent (CLI) and the API: reports, firewall parsing, port verdicts
│   ├── help/         # Meshploy explaining itself: topics/*.md, read by the console's help drawer, the docs' Concepts and (later) MCP
│   └── server/       # API core — config, service, handler, middleware, k8s (imported by apps/api)
├── deploy/           # Headscale, CoreDNS, Docker Compose infra
├── go.work           # Go Workspaces: ties apps/* + packages/*
└── .env              # Local secrets (never committed)
```

---

## Architecture overview

- **apps/api** — Thin CE entrypoint: `main.go` calls `server.Main()`. The API itself lives in `packages/server` — business logic in `service/`, HTTP concerns in `handler/`, config in `config/`.
- **apps/proxy** — Minimal L7 reverse proxy, its code in `packages/proxy` (`apps/proxy/main.go` calls `proxy.Main()`, as the API calls `server.Main()`), so an edition can build its own proxy on it. Reads the `Host` header → in-memory route cache (backed by PostgreSQL, reloaded within a second of a change through `edge_versions`, and every 30s regardless) → streams over WireGuard mesh to target node. Listens on port 8081. Once the mesh policy is enforced, an internal route answers only the machines whose owners may use what it leads to (`packages/proxy/meshgate`).
- **apps/cli** — Static Go binary (`/usr/local/bin/meshploy`). Wraps API calls and shells out to `install.sh` / `uninstall.sh` for node operations. Built with Cobra.
- **packages/db** — Shared GORM models backed by **PostgreSQL**. `AutoMigrate` + supplementary partial unique indexes run on API startup via `db.Migrate()`. Exports an Extensible Migration Registry (`RegisterMigration`) for the EE open-core pattern. Imported by both `apps/api` and `apps/proxy`.
- **packages/client** — Typed Go REST client for the Meshploy API. Imported by `apps/cli` (every command) and by `packages/mcpserver`. Lives in `packages/` so no app depends on another app.
- **packages/mcpserver** — The MCP tool definitions (~100 tools) built on `packages/client`. Imported by `apps/cli` for the local stdio server (`meshploy mcp`) and by `packages/server` for the gateway-served remote `/mcp` endpoint.
- **apps/web** — Vite + React 19 + TanStack Router frontend. Dark-only, Tailwind CSS v4 (CSS-first via `@tailwindcss/vite`, no config file), shadcn/ui Nova preset, `@base-ui/react` primitives.
- **deploy/** — Headscale (WireGuard mesh), CoreDNS, Docker Compose. The gateway node is the only public-internet-facing machine; all workers are dark.

### Help is one text

`packages/help/topics/*.md` is the only place a feature is explained to users. The console bundles it (the `virtual:help-topics` Vite plugin; the web image gets the folder as a second build context, `--build-context help=packages/help`) for its help drawer (a page's "?" or the `?` key), the ⓘ beside a term, and empty states; `apps/docs/sync-docs.mjs` publishes it as the docs' **Concepts**; the MCP server can serve it. A topic is a header (`id`, `title`, `summary`, `pages`), sections `## Title {#id}`, and a `## Terms {#terms}` glossary of `- **Label** {#term -> section}: text` that the console's tooltips show. `packages/help`'s test fails a term pointing at a missing section, or an em-dash. `CONCEPTS.md` is the architecture's design decisions, not this.

### Edge configuration is generated

`caddy/Caddyfile`, `coredns/Corefile`, the zone files and `headscale/config/config.yaml` are **output**, rendered by `meshploy domain apply` from an edge snapshot at `$HOST_DIR/state/edge.json`. Headscale's is generated because its split DNS must name every base domain's internal zone and its `server_url` follows the primary. They are not shipped in the deploy tarball and editing them is pointless - the next apply overwrites them. An operator's own Caddy configuration goes in `caddy/conf.d/*.caddy`, which the generated file imports and nothing writes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meshploy/meshploy](https://github.com/meshploy/meshploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
