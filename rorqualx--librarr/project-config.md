---
trigger: always_on
description: Project memory for Claude sessions working in this repo.
---

# CLAUDE.md

Project memory for Claude sessions working in this repo.

## Project

**Librarr** — ebook/audiobook collection manager, the OpenLibrary-based
continuation of the archived Readarr (Servarr-family sibling of Sonarr,
Radarr, Lidarr). Forked from Readarr at upstream `develop` HEAD
`0b79d300` ("Retirement announcement", 2025-06-27). Currently at
**`1.0.0-beta`** — engineering gate cleared, see
[`CHANGELOG.md`](CHANGELOG.md). The previous upstream tagged release was
`v0.4.18.2805` (commit `7cc02f95`, 2025-06-10).

Full architecture map: **`ARCHITECTURE.md`** at repo root (including a
"Librarr fork additions" section), plus per-folder `README.md` files in
every major directory.

## Identity quirk (read this first)

The csproj/assembly names are `Readarr.*` but C# **namespaces are still
`NzbDrone.*`** — set deliberately in `src/Directory.Build.props:97-99` via
a `RootNamespace` rewrite. `using Readarr.Core;` will NOT compile —
`using NzbDrone.Core;` will. Don't "fix" this; it's intentional. The
`Stylecop.ruleset:1` file even still labels itself "Rules for Radarr"
from when the ruleset was forked over.

Also intentionally kept as `readarr`/`Readarr`:

- **On-disk identifiers** in
  `src/NzbDrone.Common/Extensions/PathExtensions.cs:15-26` — `readarr.db`,
  `readarr.restore`, `readarr_update`, `readarr_backup`,
  `readarr_appdata_backup`, `Readarr.Update`. Renaming would break
  `LegacyMigrationService` (expects `readarr.db` as input) and any
  existing install upgrading in place.
- **Binary names** produced by csproj — `Readarr.exe`,
  `Readarr.Console.exe`, macOS `CFBundleExecutable=Readarr`. These
  follow the csproj names above.
- **Cross-app icons** under
  `frontend/src/Content/Images/Icons/logo-{readarr,sonarr,radarr,lidarr,prowlarr}.png`
  — these display *other* Servarr family members in the UI, not us.

## Common commands

```bash
# Frontend
yarn install                                    # install deps (root package.json)
yarn start                                      # webpack --watch
yarn build                                      # one-shot → _output/UI
yarn lint                                       # ESLint
yarn stylelint-linux                            # Stylelint over CSS

# Backend
./build.sh --backend --enable-extra-platforms   # full backend build (multi-RID)
./test.sh                                       # backend tests
dotnet test src/NzbDrone.Core.Test/             # one test project
dotnet run --project src/NzbDrone.Console/      # run on :8787 (HTTPS :6868)

# Single test
dotnet test src/NzbDrone.Core.Test/ --filter "FullyQualifiedName~MyClassTests"
```

CI uses the same scripts. **Note:** StyleCop only enabled on the Linux CI
leg (`azure-pipelines.yml:79`); Mac/Windows skip it.

## Stack (develop HEAD versions)

- **Backend:** .NET 6 (`dotnetVersion: '6.0.427'`), ASP.NET Core, **DryIoc
  5.4.3** DI (`src/NzbDrone.Host/Bootstrap.cs:9-10,90`), custom Dapper-based
  ORM in `NzbDrone.Core/Datastore/`, Servarr-forked FluentMigrator
  (41 migrations), dual **SQLite + PostgreSQL**, NLog logging. Sentry 3.31
  for error reporting. Shipping version `1.0.0-beta`
  (`azure-pipelines.yml:22`). `Directory.Build.props:77`
  `AssemblyVersion 10.0.0.*` is the historical Readarr placeholder
  the CI overwrites at build time; not the shipping version.
- **Frontend:** React 17 + Redux 4 (**legacy `createStore`**, not RTK),
  Webpack 5, CSS Modules via PostCSS, `@microsoft/signalr`. Partial
  JS→TS migration (~985 `.js` / 375 `.ts` / 36 `.tsx` — ~29% TS).
  ~151 hook callsites alongside dominant class-component style.
- **Tests:** NUnit + Moq + FluentAssertions. Selenium 3.141 + ChromeDriver
  91 in `NzbDrone.Automation.Test` (years out of date — treat that suite
  as historical).

## Conventions

- **Strict build:** `TreatWarningsAsErrors=true`,
  `EnforceCodeStyleInBuild=true` (`src/Directory.Build.props:4-5`).
- **Backend file layout** per domain under `NzbDrone.Core/{Domain}/`:
  `Model/`, `{Entity}Repository.cs`, `{Entity}Service.cs`, `Commands/`,
  `Events/` (with `Handlers/`).
- **REST:** `Readarr.Api.V1/{Domain}/{Entity}Controller.cs` +
  `{Entity}Resource.cs` DTO. Manual mapping (no AutoMapper).
- **Frontend:** PascalCase folder per component, `Foo.js` + `Foo.css` +
  `FooConnector.js`. PropTypes for `.js`, TS types for `.ts/.tsx`
  (`react/prop-types: 2 / off` in `frontend/.eslintrc.js:317,365`).
- **Provider plugins** (indexers, download clients, notifications, import
  lists, metadata): all derive from a `ThingiProvider`-rooted base and
  are auto-discovered by DryIoc reflection — no manual registration.
- **Messaging:** `EventAggregator` (in-process pub/sub) + `CommandQueueManager`
  (DB-backed background queue). Handlers implement `IHandle<TEvent>` /
  `IExecute<TCommand>`.

## Where to add things

| Task | Location |
|---|---|
| New indexer | `src/NzbDrone.Core/Indexers/` — extend `HttpIndexerBase` |
| New download client | `src/NzbDrone.Core/Download/Clients/` — extend `TorrentClientBase` or `UsenetClientBase` |
| New notification | `src/NzbDrone.Core/Notifications/` — extend `NotificationBase` |
| New import list | `src/NzbDrone.Core/ImportLists/` — extend `HttpImportListBase` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rorqualx/Librarr](https://github.com/Rorqualx/Librarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
