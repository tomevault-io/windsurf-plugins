---
trigger: always_on
description: validates server-side configured credentials and issues a short-lived HS256 JWT
---

# AGENTS.md — Vibe Code Stack For CEOs

AI-first Turborepo monorepo (pnpm 11, Node 22, TypeScript strict): 3 frontend
apps + 3 backend services + 3 shared packages, deployed to Cloudflare and
private AWS EC2 infrastructure.

- This file is the single source of truth for agent behavior. `CLAUDE.md` is a
  symlink to it — always edit `AGENTS.md`.
- This is intentionally the ONLY `AGENTS.md` in the repo (no nested
  per-workspace files) — rules are scoped per workspace inside this file to
  avoid drift. Don't create nested agent files.
- When a rule here conflicts with existing code, follow the rule and flag the
  code.

## Monorepo map

| Workspace | Package | What it is | Deploys to |
|-----------|---------|------------|------------|
| `apps/dapp` | `@apps/dapp` | Next.js 16 App Router (vinext/Vite) | Cloudflare Workers |
| `apps/admin` | `@apps/admin` | React 19 SPA (Rsbuild + React Router, no RSC) | Cloudflare Pages |
| `apps/landing` | `@apps/landing` | Astro static site (zero JS by default) | Cloudflare Workers |
| `services/trading-rpc` | `@services/trading-rpc` | NestJS host on Fastify: Connect for edge + native Nest gRPC for Node services | AWS EC2 (Docker/ECR/SSM) |
| `services/admin-rpc` | `@services/admin-rpc` | Admin-facing NestJS RPC facade; calls trading-rpc over native gRPC for coin data | AWS EC2 (Docker/ECR/SSM) |
| `services/api-gateway` | `@services/api-gateway` | Edge gateway Worker (Hono: request-id, CORS, self-hosted Durable Object rate-limit, opt-in JWT auth, upstream proxy) | Cloudflare Workers |
| `packages/protocol` | `@packages/protocol` | Protobuf schemas, buf codegen → `src/gen/` | — |
| `packages/api-core` | `@packages/api-core` | Shared RPC impl + CORS-aware fetch handler | — |
| `packages/api-client` | `@packages/api-client` | Typed Connect-RPC browser client | — |

Rule scope: Server/Client Component rules apply to `apps/dapp` only. All three
frontend apps use an FSD-inspired layered architecture with explicit
`bootstrap`/`screens` names and framework-specific entrypoints documented below.
The backend slice architecture applies to `packages/api-core` + `services/*`
(see Architecture rules).
Everything else (naming, testing, git, security) applies repo-wide.

## Tech stack (mind the major versions — APIs differ across generations)

| Layer | Tool + version |
|-------|----------------|
| Framework (dapp) | Next.js 16 App Router on vinext 0.1 (Vite) |
| UI | React 19 · Panda CSS 1.x + Ark UI 5 (headless) |
| Language | TypeScript 6, `strict: true` |
| Validation | Zod **4** (not v3 — different error/message APIs) |
| Server state | TanStack Query 5 |
| Client/URL state | Zustand 5 · nuqs 2 |
| Forms | react-hook-form 7 + Zod resolver |
| Server actions | next-safe-action **8** |
| Tables | TanStack Table 8 |
| HTTP | ofetch 1 (via shared `xhr`) · Connect-RPC 2 (`@connectrpc/*`) |
| API server (Node) | NestJS 11 + Fastify 5 + ConnectRPC 2 + native Nest gRPC · edge gateway on Hono 4 |
| Database | PostgreSQL 18 · Drizzle ORM 0.45 + Drizzle Kit 0.31 on node-postgres 8 |
| Auth | iron-session 8 (encrypted cookies) |
| Admin | Rsbuild 2 (Rspack) · React Router **7** |
| Landing | Astro 7 |
| Testing | Vitest 4 + Testing Library + MSW 2 + Playwright |
| Lint/format | Biome 2 + ESLint 10 (flat config) + buf |
| Monorepo | Turborepo 2 + pnpm 11 workspaces |

## Commands

```bash
mise run setup                # install locked tools + frozen dependencies

mise run dev                  # native apps + managed PostgreSQL/VPC infra
mise run dev:web | dev:admin | dev:landing          # one frontend
mise run dev:api | dev:admin-api | dev:gateway | dev:backend  # backend topology
mise run dev:infra:stop       # stop native-development Docker infra

mise run typecheck            # tsc --noEmit, all 9 workspaces
mise run check:ci             # Biome (read-only), whole repo
mise run lint                 # ESLint / Biome / buf / architecture checks
mise run test                 # toolchain tests + Vitest, all workspaces
mise run test:coverage        # enforce dapp/admin logic coverage thresholds
mise run build                # production builds
mise run check                # Biome auto-fix + format
mise run verify               # all definition-of-done gates, sequentially
mise run test:docker          # Docker builds + PostgreSQL backup/restore integration
mise run test:protocol        # codegen drift + protobuf breaking check
mise run security:audit       # high-severity dependency audit

mise run docker:start             # full Docker development stack
mise run docker:start:dapp | docker:start:admin | docker:start:landing
mise run docker:start:api-gateway | docker:start:admin-rpc | docker:start:trading-rpc
mise run docker:stop | docker:check
mise run terraform:check       # fmt + provider-backed validate; never apply

# pnpm is internal; use it directly only for targeted commands without a mise task.
pnpm --filter @apps/dapp test                                   # one workspace
pnpm --filter @apps/dapp exec vitest run <path-to-test-file>    # one test file
mise run test:e2e             # Playwright (apps/dapp/e2e/); needs browsers installed
```

## Definition of done

Run these before declaring any task complete. CI runs exactly the same gates.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NoahDuongMaster/nextjs-boilerplate](https://github.com/NoahDuongMaster/nextjs-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
