---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**Contentment for Umbraco** — a community package distributing property editors, data sources, and related components for Umbraco CMS. Ships as the `Umbraco.Community.Contentment` NuGet package.

The repository is a **two-project package**:

- `src/Umbraco.Community.Contentment/` — C# Razor class library (`Microsoft.NET.Sdk.Razor`), multi-targeting `net9.0` (Umbraco v16) and `net10.0` (Umbraco v17). Produces the NuGet package.
- `src/Umbraco.Community.Contentment.Client/` — TypeScript/Lit Umbraco backoffice extension (ES module library). Built output is emitted into the C# project's `wwwroot/App_Plugins/Contentment/` so it ships inside the NuGet package as a static web asset. Client-specific conventions live in `.claude/rules/client.md`.

The `src/Umbraco.Cms.16.x/` and `src/Umbraco.Cms.17.x/` folders are **demo/runtime host sites** used for local development and manual testing (configured for unattended install with SQLite and uSync). They are not part of the shipped package.

## Branching

The active development branch is **`contrib`** (v6.x, current). Each major Contentment version has its own long-lived `dev/vX.x` branch that targets specific Umbraco versions — check `.github/CONTRIBUTING.md` for the full mapping before backporting.

## Common Commands

Run from the repo root unless noted.

### Client (TypeScript/Lit)

```bash
cd src/Umbraco.Community.Contentment.Client
npm install
npm run dev                  # tsc + vite build --watch (active development)
npm run build                # tsc + vite build + tsc types (production package + .d.ts)
```

Node >= 22 (pinned in `.nvmrc`). The build writes directly into `../Umbraco.Community.Contentment/wwwroot/App_Plugins/Contentment/`.

`npm run generate:server-api` regenerates the OpenAPI client under `src/api/` from `http://localhost:21187/umbraco/swagger/contentment/swagger.json` — this requires a running Umbraco host (e.g. `Umbraco.Cms.17.x`) on that port. Don't run it without the host up.

### Server (C# package)

```bash
# Pack the NuGet (matches what CI does, minus versioning)
build/build-pkgs.cmd
# …which runs:
dotnet pack src/Umbraco.Community.Contentment/Umbraco.Community.Contentment.csproj -c Release -o artifacts
```

The release workflow (`.github/workflows/release.yml`, triggered by a SemVer tag push) builds the client first, then `dotnet pack` with `/p:Version=${tag}`, then pushes to GitHub Packages + NuGet.org.

### Local run / manual testing

Run one of the host projects (e.g. `dotnet run --project src/Umbraco.Cms.17.x`). The host references `Umbraco.Community.Contentment.csproj` via `ProjectReference`, so changes rebuild into the host on run. The client must be rebuilt separately (use `npm run dev` in parallel).

**There is no automated test suite** — verification is done by running a host site and exercising the editors in the backoffice.

## Architecture

### C# project layout (`src/Umbraco.Community.Contentment/`)

- `DataEditors/{EditorName}/` — one folder per property editor (e.g. `TextInput/`, `ContentBlocks/`, `DataList/`). Each typically contains a `{Name}DataEditor.cs` (the `[DataEditor]`-attributed class), a `ValueConverter`, and sometimes configuration field classes.
- `DataEditors/_/` — shared interfaces (`IContentmentDataSource`, `IContentmentListItem`, `IContentmentDisplayMode`, etc.) plus shared `ConfigurationFields/`, `DataSources/`, and `DataValueEditors/`.
- `Composing/ContentmentComposer.cs` — single `IComposer` wiring up services, custom collection builders (`ContentmentListItemCollectionBuilder`, `ContentmentDataListItemPropertyValueConverterCollectionBuilder`), config binding, and notification handlers.
- `Api/Management/` — backoffice Management API controllers (Umbraco 14+ style). Exposed under the `contentment` Swagger group (see `ContentmentOperationIdHandler`, `ConfigureContentmentSwaggerGenOptions`).
- `ContentmentPackageManifestReader.cs` — registers the client bundle by emitting a package manifest that points at `App_Plugins/Contentment/*`.
- `Constants.cs` / `ContentmentConstants.cs` — `Constants.Internals` holds all alias/path/namespace prefixes; `ContentmentConstants` is the public surface (editor aliases, config keys). New editors should follow the existing naming pattern (`DataEditorAliasPrefix + "Name"`, UI alias `Umb.Contentment.PropertyEditorUi.Name`).
- `Migrations/` — `ContentmentPlan` chains install/upgrade migrations.
- `Notifications/` — `INotificationHandler` implementations (telemetry, data type save/delete, content blocks copy).
- `Services/ContentmentContentContext.cs` — abstraction for the current content context used by editors.

### Client

Client conventions, layout, naming rules, and the list of extension types implemented are in `.claude/rules/client.md` — path-scoped to the client project so it auto-loads only when Claude reads files under `src/Umbraco.Community.Contentment.Client/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leekelleher/umbraco-contentment](https://github.com/leekelleher/umbraco-contentment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
