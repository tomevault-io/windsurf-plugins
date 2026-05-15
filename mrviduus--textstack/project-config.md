---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Free book library w/ Kindle-like reader. Upload EPUB/PDF/FB2 → parse → SEO pages + offline-first sync.

**Live**: [textstack.app](https://textstack.app/) (public) · [textstack.dev](https://textstack.dev/) (admin)

**Stack**: ASP.NET Core (API + Worker) + PostgreSQL + React + React Native (Expo)

**Prerequisites**: Docker, .NET 10 SDK, Node.js 18+, pnpm

**CI/CD**: Push to `main` → auto-deploy. SSG rebuild: admin panel or `make rebuild-ssg`.

## Commands

```bash
# Setup (one-time)
cp .env.example .env          # Edit with real values
make nginx-setup              # Install nginx config (Linux)
make nginx-setup-mac          # Mac
make up                       # Start services

# Docker
make up / down / restart / logs / status
make build                    # docker compose up -d --build
make rebuild                  # full rebuild --no-cache
make clean-ssg                # remove dist/ssg*
make fix-permissions          # Fix volume permissions
make reindex-search           # Rebuild search indexes

# Deploy
make deploy                   # Full deploy (pull, build, restart, SSG)
make rebuild-ssg              # Rebuild SSG pages only

# Database
make backup                   # Backup to ~/backups/textstack/
make backup-list              # List all backups
make restore FILE=path.gz     # Restore from backup
docker compose exec db psql -U app books   # DB shell
docker compose down -v                      # Reset all (loses data)

# Tests
dotnet test                                 # All tests
dotnet test tests/TextStack.UnitTests
dotnet test tests/TextStack.IntegrationTests
dotnet test tests/TextStack.Extraction.Tests
dotnet test tests/TextStack.Search.Tests
dotnet test --filter "Name~TestMethodName"  # Single test
pnpm -C apps/web test                       # Frontend unit tests (Vitest)
pnpm -C apps/web test:watch                 # Watch mode
pnpm -C apps/web test:e2e                   # Playwright E2E (headless)
pnpm -C apps/web test:e2e:ui                # Playwright E2E (UI mode)

# Lint
dotnet format textstack.sln                  # Backend

# CLI commands (via dotnet run --project backend/src/Api --)
# create-admin <email> <password> [role]
# optimize-images [--dry-run]
# import-textstack <book-path>

# Local dev (no Docker)
dotnet run --project backend/src/Api
dotnet run --project backend/src/Worker
pnpm -C apps/web dev          # http://localhost:5173
pnpm -C apps/admin dev        # http://localhost:81

# Build
pnpm -C apps/web build
pnpm -C apps/admin build

# Migrations
dotnet ef migrations add <Name> --project backend/src/Infrastructure --startup-project backend/src/Api
MIGRATE_TARGET=0 docker compose up migrator   # Rollback all migrations

# Mobile app (apps/mobile)
cd apps/mobile
npx expo start                    # Dev server (Expo Go — limited native modules)
npx expo run:ios                  # Local iOS build (requires Xcode)
npx expo run:android              # Local Android build (requires Android Studio)
npx tsc --noEmit                  # TypeScript check
npm run build:dev:ios             # EAS dev build (cloud, requires eas login)
npm run build:prod                # EAS production build
npm run submit:ios                # Submit to App Store
npm run submit:android            # Submit to Google Play
```

| Service | Local | Prod |
|---------|-------|------|
| Web | http://localhost:5173 | https://textstack.app |
| API | http://localhost:8080 | https://textstack.app/api |
| API Docs | http://localhost:8080/scalar/v1 | — |
| Admin | http://localhost:81 | https://textstack.dev |
| Aspire | http://127.0.0.1:18888 | — |

**Storage**: Files at `./data/storage/books/{editionId}/` (originals + derived covers).

## Architecture

```
API → Application → Domain ← Infrastructure
                      ↑
                   Worker
```

- **Domain**: Pure C#, no framework deps
- **Application**: Business logic, interfaces (`IAppDbContext`, `IFileStorageService`)
- **Contracts**: Shared DTOs (request/response models) used by API and Application
- **Infrastructure**: EF Core (snake_case naming), storage implementations
- **API/Worker**: Orchestration, DI

**Middleware pipeline** (order matters): `ForwardedHeaders` → `Cors` → `RateLimiter` → `ExceptionMiddleware` → `StaticFiles(/storage)` → `/health` → `SiteContext` → `LanguageContext` → `GuestActivity` (LastActiveAt debounce hourly) → `Routing` → `AdminAuth` (conditional on `/admin/*`)

**Site resolution**: Single-site now (ADR-007). `SiteContextMiddleware` still resolves host → SiteId. Dev mode: `?site=` query param override.

**Patterns**:
- Endpoints: `Map{Domain}Endpoints()` in `Api/Endpoints/`
- Test naming: `{Method}_{Scenario}_{Expected}`

### Frontend Architecture

**No Redux/Zustand** — React Context only. Provider hierarchy in `App.tsx`:
```
BrowserRouter → SiteProvider → AuthProvider → GuestLimitsProvider → NativeLanguageProvider → DownloadProvider → AppRoutes
  └─ /:lang/* → LanguageProvider → Header + page routes
```

- **SiteProvider**: Fetches `/api/site/context`, provides `site` to all children

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrviduus/textstack](https://github.com/mrviduus/textstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
