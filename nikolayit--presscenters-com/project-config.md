---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

PressCenters.com is an ASP.NET Core news aggregator that scrapes press releases from Bulgarian
government sites, institutions, NGOs, and state companies, then republishes them. Content is in
Bulgarian. The codebase follows the SoftUni "Simple Web Template" layered architecture.

## Commands

All commands are run from the repository root. The solution lives in `src/`.

```bash
# Build the whole solution
dotnet build src/PressCenters.sln

# Run all tests
dotnet test src/PressCenters.sln

# Run tests for a single project
dotnet test src/Tests/PressCenters.Services.Sources.Tests

# Run a single test class or method (xUnit filter)
dotnet test src/Tests/PressCenters.Services.Sources.Tests --filter "FullyQualifiedName~BnbBgSourceTests"
dotnet test src/Tests/PressCenters.Services.Sources.Tests --filter "FullyQualifiedName~BnbBgSourceTests.GetNewsShouldReturnResults"

# Run the web app (requires SQL Server + a DefaultConnection connection string)
dotnet run --project src/Web/PressCenters.Web

# Run the Sandbox console app (ad-hoc scraping, backfills, one-off maintenance)
dotnet run --project src/Tests/Sandbox
```

All projects target **net10.0**. Visual Studio 2022 is the primary IDE (`.sln`, `.csproj.user`,
`.vs/` present). CI runs on GitHub Actions (`.github/workflows/ci.yml`): `dotnet build` + `dotnet test` of
`src/PressCenters.sln` in Release on `windows-latest` (Windows keeps the legacy code-page encodings
the scrapers depend on). There is also a CodeQL workflow in `.github/workflows/`.

## Architecture

### Layered structure (`src/`)

- **PressCenters.Common** — cross-cutting helpers: `GlobalConstants` (system name, default user-agent),
  `ReflectionHelpers.GetInstance<T>(typeName)`, `StringExtensions`.
- **Data/** — `PressCenters.Data.Models` (EF entities), `PressCenters.Data` (`ApplicationDbContext`,
  migrations, repositories, seeding), `PressCenters.Data.Common` (base model types + repository interfaces).
- **Services/** — `PressCenters.Services` (DTOs like `RemoteNews`, `SlugGenerator`),
  `PressCenters.Services.Data` (business services over repositories), `PressCenters.Services.Sources`
  (the scrapers — the heart of the app), `PressCenters.Services.CronJobs` (Hangfire jobs),
  `PressCenters.Services.Mapping` (reflection-based Mapster), `PressCenters.Services.Messaging` (email).
- **Web/** — `PressCenters.Web` (MVC app, admin area, Hangfire wiring in `Startup.cs`) and
  `PressCenters.Web.Infrastructure`.
- **Tests/** — xUnit test projects + the `Sandbox` console runner.

### The scraping system (most important concept)

Each news provider is a class deriving from `BaseSource` (`Services/PressCenters.Services.Sources/`),
organized into folders by category: `BgInstitutions`, `Ministries`, `Municipalities`, `BgNgos`,
`BgStateCompanies`. A source implements:

- `BaseUrl` — the site root.
- `GetLatestPublications()` — fetch the listing page, select article links, call `GetPublication(url)`
  on each. The `BaseSource.GetPublications(address, anchorSelector, ...)` helper does this generically.
- `ParseDocument(IDocument, url)` — parse one article into a `RemoteNews` (title, HTML content, post
  date, image URL). Uses **AngleSharp** for HTML parsing/CSS selectors.
- Optional: `GetAllPublications()` for full backfills, `ExtractIdFromUrl(url)` override,
  `UseProxy => true` (routes fetches through a random relay host via
  `PressCenters.Common.ProxyUrlBuilder` over `GlobalConstants.ProxyHosts`), custom `Encoding` or `Headers`.

`BaseSource.GetPublication()` orchestrates fetch → parse → normalize (trims title/content, clamps
future dates, normalizes image URL, sets `OriginalUrl`, derives `RemoteId` via `ExtractIdFromUrl`,
which defaults to the last URL path segment).

`RemoteNews` (the scraper DTO in `PressCenters.Services`) is distinct from `News` (the EF entity).
`NewsService.AddAsync` maps one to the other and **deduplicates on `(SourceId, RemoteId)`** — if a
row with the same remote id already exists for that source, it is skipped.

`MainNews` is a separate, simpler concept: homepage "top story" providers derive from
`BaseMainNewsProvider` (`Sources/MainNews/`, e.g. BTA, CNN, Reuters, Nova) and return a single
`RemoteMainNews` headline.

### Source registration & scheduling (reflection-driven)

Sources are **not** auto-discovered by type scanning. They are listed in
`Data/PressCenters.Data/Seeding/SourcesSeeder.cs` as rows keyed by `TypeName` (the fully-qualified
class name). On startup `Startup.Configure` runs `dbContext.Database.Migrate()`, seeds, then
`SeedHangfireJobs` registers one recurring Hangfire job **per DB source row**:

- `GetLatestPublicationsJob` per source — every 7 minutes; resolves the source instance with
  `ReflectionHelpers.GetInstance<BaseSource>(typeName)`, fetches latest, adds new ones, downloads images.
- `MainNewsGetterJob` — every 2 minutes.
- `DbCleanupJob` — weekly.

The Hangfire dashboard is at `/hangfire` (production only, Administrator role required).

### Images

`NewsService.SaveImageLocallyAsync` downloads each article image and writes two resized PNGs via

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NikolayIT/PressCenters.com](https://github.com/NikolayIT/PressCenters.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
