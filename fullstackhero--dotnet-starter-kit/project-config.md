---
trigger: always_on
description: > A production-ready modular .NET 10 monolith + two React 19 apps, built for enterprise SaaS.
---

# FullStackHero .NET Starter Kit

> A production-ready modular .NET 10 monolith + two React 19 apps, built for enterprise SaaS.

This file is the canonical guide for **all** AI coding tools (Claude Code, Gemini CLI, Cursor, Codex, …).
`CLAUDE.md` and `GEMINI.md` are thin bridges that import this file — edit conventions **here**, not there.

This file is the map. Detailed conventions live in `.agents/rules/` and are read on demand — **read the
relevant rule file before working in that area** (see the index below). Keep this file lean.

## What this is

A **modular monolith** (Vertical Slice Architecture) backend that ships with two **React + Vite**
front-ends and a CLI. Multitenancy, auth, auditing, billing, files, chat and more are first-class.

- **Backend** — .NET 10, EF Core 10, PostgreSQL, Redis, JWT + ASP.NET Identity, Finbuckle multitenancy,
  Hangfire, OpenAPI/Scalar, Serilog + OpenTelemetry, .NET Aspire.
- **Frontends** — `clients/admin` (operator-facing) and `clients/dashboard` (tenant-facing): React 19,
  Vite 7, TypeScript, TanStack Query v5, React Router 7, Radix + Tailwind v4 (shadcn-style), SignalR/SSE.

## Repo map

| Path | What |
|------|------|
| `src/BuildingBlocks/` | Shared framework libraries (Core, Persistence, Web, Caching, Eventing, Storage, Quota…). **Protected — see below.** |
| `src/Modules/{Name}/` | Bounded contexts. Each has a runtime project + a `.Contracts` project (its only public API). |
| `src/Host/FSH.Starter.Api` | Composition-root Web API host. |
| `src/Host/FSH.Starter.AppHost` | .NET Aspire orchestrator (Postgres, Redis, MinIO, migrator, API, **both React apps**). |
| `src/Host/FSH.Starter.DbMigrator` | One-shot migrate/seed runner. DB is **not** migrated at API startup. |
| `src/Host/FSH.Starter.Migrations.PostgreSQL` | All EF migrations, organized per-module by folder. |
| `src/Tests/` | Per-module tests, `Architecture.Tests` (NetArchTest), `Integration.Tests` (Testcontainers). |
| `src/Tools/CLI` | The `fsh` CLI (Spectre.Console). |
| `clients/admin`, `clients/dashboard` | The two React apps. |
| `deploy/` | Infra (docker, terraform, dokploy). |

## Tech stack

| Backend | | Frontend | |
|---|---|---|---|
| Framework | .NET 10 / C# latest | Framework | React 19 + Vite 7 + TS 5.x |
| CQRS | Mediator 3.x (source-gen) | Data | TanStack Query v5 |
| Validation | FluentValidation 12.x | Routing | React Router 7 |
| ORM / DB | EF Core 10 / PostgreSQL (Npgsql) | UI | Radix + Tailwind v4 + CVA (shadcn) |
| Auth | JWT Bearer + ASP.NET Identity | Forms | react-hook-form + zod (**admin only**) |
| Multitenancy | Finbuckle 10.x | Realtime | `@microsoft/signalr`, SSE (dashboard) |
| Cache / Jobs | Redis, Hangfire | Tests | Playwright (route-mocked) |
| Docs | OpenAPI + Scalar | API client | hand-written `apiFetch` (no codegen) |
| Hosting | .NET Aspire | Env | runtime `/config.json` (not `VITE_*`) |
| Testing | xUnit, Shouldly, NSubstitute, AutoFixture, NetArchTest, Testcontainers | | |

## Build & run

```bash
# Whole stack (Postgres + pgAdmin + Redis + MinIO + migrator + API + both React apps)
dotnet run --project src/Host/FSH.Starter.AppHost   # one-time: npm install in clients/admin & clients/dashboard

dotnet build src/FSH.Starter.slnx                   # build backend
dotnet run --project src/Host/FSH.Starter.Api       # API only → https://localhost:7030 (/scalar)
dotnet test src/FSH.Starter.slnx                    # tests — integration tests REQUIRE Docker

cd clients/admin && npm install && npm run dev       # → http://localhost:5173
cd clients/dashboard && npm install && npm run dev   # → http://localhost:5174
```

Migrations / seed (DbMigrator, separate step):
```bash
dotnet run --project src/Host/FSH.Starter.DbMigrator -- apply [--seed]
dotnet run --project src/Host/FSH.Starter.DbMigrator -- list-pending
```

**Ports:** API 7030 (https)/5030 (http) · admin 5173 · dashboard 5174 · Postgres 5432 · pgAdmin 5050 · Valkey 6379 · MinIO 9000/9001.

## Branching & PRs

Single long-lived branch: **`main`** (the default) — there is **no `develop`**. Branch from and target `main`; stable releases are cut from `v*` tags. CI is split into path-scoped **Backend CI** (`src/**`) and **Frontend CI** (`clients/**`) workflows; branch protection requires only those two gate checks — never the individual jobs, which are skipped on the other side's PRs.

## Golden rules (do not break)

1. **Module boundaries** — a module references another module only through its `.Contracts` project, never its runtime project. Enforced by `Architecture.Tests`.
2. **Registering a module touches FOUR places** — `Program.cs` Mediator `o.Assemblies` (two markers each) + `moduleAssemblies` array, **and the identical pair in `DbMigrator/Program.cs`**. A missing Mediator marker = handlers silently undiscovered. See `architecture.md`.
3. **Tenant isolation is default-ON** via `BaseDbContext`. Opt out only via `IGlobalEntity`. Subclass DbContexts call `base.OnModelCreating` **last**. See `database.md`.
4. **Do NOT modify `src/BuildingBlocks`** without explicit approval — shared by every module, wide blast radius.
5. **Mediator handlers must be `public sealed`**, return `ValueTask<T>`, and `.ConfigureAwait(false)` every await.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fullstackhero/dotnet-starter-kit](https://github.com/fullstackhero/dotnet-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
