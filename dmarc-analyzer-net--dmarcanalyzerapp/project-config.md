---
trigger: always_on
description: Orientation for AI coding agents (and new contributors) working in **DmarcAnalyzerApp** — an agency-first, self-hosted DMARC analyzer (ASP.NET Core + Carter API, React + Vite frontend, PostgreSQL). Read this first, then follow the links into the detailed docs.
---

# AGENTS.md

Orientation for AI coding agents (and new contributors) working in **DmarcAnalyzerApp** — an agency-first, self-hosted DMARC analyzer (ASP.NET Core + Carter API, React + Vite frontend, PostgreSQL). Read this first, then follow the links into the detailed docs.

## What this project is

One agency workspace monitors DMARC aggregate (RUA) reports for many clients across many domains. Reports are pulled from mailboxes over IMAP, parsed, stored, and surfaced as compliance analytics with per-source drill-down. Multi-tenant (client-scoped), role-gated, and packaged as a single container.

- Product overview & local/Docker run: [`README.md`](README.md)
- Current implementation snapshot (what's actually built): [`docs/planning/status.md`](docs/planning/status.md)
- Prioritized work + recommended sequence: [`docs/planning/backlog.md`](docs/planning/backlog.md)

## Repository layout

- `src/api` — backend. One image and one entrypoint (`Program.cs`), running in the mode `APP_MODE` selects: `api` (REST API + the built React app from `wwwroot`), `worker` (mailbox-sync host), `all` (both in one process), or `migrate` (apply migrations and exit). Backend notes: [`src/api/README.md`](src/api/README.md).
  - `Application/` — service layer (Auth, Analytics, Clients, Domains, MailboxSources, Ingestion, Reports, Security, Users). Carter modules in `Modules/` are thin and delegate here.
  - `Data/` — EF Core `DmarcAnalyzerDbContext`, entities, and `Migrations/`. A design-time factory (`DmarcAnalyzerDbContextFactory`) lets `dotnet ef` run without building the web host.
  - `Middleware/` — `SessionAuthMiddleware` (cookie session → `ICurrentUserContext`) then `RoleAuthorizationMiddleware` (endpoint role enforcement).
- `src/web` — React 19 + Vite + TypeScript + Tailwind v3. Pages in `src/pages`, primitives in `src/components/ui` + `src/components/data`, shared helpers in `src/lib`. Frontend notes: [`src/web/README.md`](src/web/README.md).
- `src/api.tests` — xUnit tests (EF Core InMemory provider; note raw-SQL paths can't run under InMemory).
- `src/api.integration.tests` — the tests that need a real database: raw SQL, `ON CONFLICT` dedup, transaction boundaries. Starts a PostgreSQL container via Testcontainers, so it needs Docker and takes seconds rather than milliseconds. Kept separate so `dotnet test src/api.tests` stays instant and runs with no daemon.
- `http/api.http` — REST Client request collection for manual API calls.
- `docs/` — see the doc map below.

## Build, test, run

```bash
# Backend build + tests (from repo root)
dotnet build DmarcAnalyzerApp.slnx       # or: dotnet build src/api/DmarcAnalyzer.Api.csproj
dotnet test src/api.tests                 # fast; InMemory provider
dotnet test src/api.integration.tests     # needs Docker; starts PostgreSQL

# Frontend (from src/web)
npm install
npm run build     # tsc -b && vite build   (must pass)
npm test          # vitest run              (must pass; runs in CI)
npm run lint      # eslint .                (must be clean)
npm run dev       # Vite dev server, proxies /api to the local API

# EF Core migrations (from repo root)
dotnet ef migrations add <Name> --project src/api/DmarcAnalyzer.Api.csproj

# Full dev stack in Docker (api + worker + postgres + dev Zitadel)
# Note: the repo-root compose file is the *development* stack and keeps api and
# worker separate. deploy/compose.yml is the shipped one and runs APP_MODE=all.
# Once per clone: the mailbox-credential encryption key is not committed, so
# compose refuses to start until .env has one. Keep it — changing the key makes
# stored mailbox passwords undecryptable.
echo "DMARC_ENCRYPTION_KEY=$(openssl rand -base64 32)" > .env
docker compose up -d --build
```

**Local dev URLs:** frontend `http://localhost:5173`, API `http://localhost:5076` (Vite proxies `/api`). **Docker:** API on `http://localhost:8080`, Postgres `localhost:5432`. (You can override host ports locally with a gitignored `docker-compose.override.yml`.)

**If the image build fails at `dotnet restore` with NU1004**, the lockfile and the csproj disagree — you changed a `PackageReference` and `src/api/packages.lock.json` was not regenerated. The image restores in locked mode on purpose, so that a stale lockfile stops the build rather than quietly resolving a different dependency graph into the artifact people run. NU1004 offers two remedies and the first one is wrong here: do **not** disable `RestoreLockedMode` to get past it, that removes the check instead of fixing what it caught. Regenerate and commit:

```bash
dotnet restore src/api/DmarcAnalyzer.Api.csproj --force-evaluate
```

A local `dotnet build`/`dotnet test` will *not* reproduce this — `RestoreLockedMode` is on for CI and inside the image, not for your working tree (see `src/Directory.Build.props`), so drift is invisible until one of those two runs.

### Working in a worktree

A fresh worktree has none of this repo's gitignored files, and two of them gate whether it can build or run at all:

```bash
cp ../DmarcAnalyzerApp/.env .          # or wherever the original checkout is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmarc-analyzer-net/DmarcAnalyzerApp](https://github.com/dmarc-analyzer-net/DmarcAnalyzerApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
