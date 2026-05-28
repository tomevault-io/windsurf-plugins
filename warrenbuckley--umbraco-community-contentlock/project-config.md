---
trigger: always_on
description: **Umbraco Community ContentLock** is an open-source NuGet package for **Umbraco CMS 17** (Bellissima backoffice) that prevents content editing conflicts. Editors can lock a content node while editing; locked nodes become read-only for everyone else, and publish/save/unpublish actions are hidden for other users. Real-time lock state is pushed to all connected backoffice users via **SignalR**.
---

# CLAUDE.md — Umbraco.Community.ContentLock

## Project Overview

**Umbraco Community ContentLock** is an open-source NuGet package for **Umbraco CMS 17** (Bellissima backoffice) that prevents content editing conflicts. Editors can lock a content node while editing; locked nodes become read-only for everyone else, and publish/save/unpublish actions are hidden for other users. Real-time lock state is pushed to all connected backoffice users via **SignalR**.

- NuGet package ID: `Umbraco.Community.ContentLock`
- Current version: `17.0.0`
- Target framework: `net10.0`
- Umbraco CMS dependency: `17.0.0`
- License: MIT

## Repository Structure

```
/
├── ContentLock/                  # Main package project (C# + TypeScript)
│   ├── Client/                   # TypeScript frontend (Vite, Lit web components)
│   │   ├── src/                  # Frontend source
│   │   │   ├── api/              # Auto-generated OpenAPI TypeScript client (hey-api)
│   │   │   ├── bundle.manifests.ts  # Root manifest bundle
│   │   │   ├── conditions/       # Umbraco extension conditions
│   │   │   ├── dashboards/       # Content Lock dashboard (overview of all locks)
│   │   │   ├── entityActions/    # Lock/Unlock entity actions in tree/actions menu
│   │   │   ├── entitySigns/      # Visual lock indicator on tree nodes
│   │   │   ├── entrypoints/      # Extension entry point (injects conditions into core actions)
│   │   │   ├── globalContexts/   # SignalR context (real-time state)
│   │   │   ├── headerApps/       # Online users count in backoffice header
│   │   │   ├── localizations/    # i18n translations (en, cy, dk, fr, it, nl, no, tr)
│   │   │   ├── modals/           # Online users modal
│   │   │   ├── userpermissions/  # "Unlocker" permission definition
│   │   │   ├── workspaceActions/ # "Preview Only" workspace action for locked nodes
│   │   │   ├── workspaceContexts/# Workspace-level lock context
│   │   │   └── workspaceFooterApp/ # Footer message when a node is locked
│   │   ├── package.json
│   │   └── vite.config.ts        # Builds to ../wwwroot/App_Plugins/ContentLock
│   ├── Composers/                # Umbraco IComposer DI registrations
│   ├── Controllers/              # ContentLockApiController (Lock/Unlock/BulkUnlock)
│   ├── Extensions/               # IUser extensions, UmbracoBuilder SignalR setup
│   ├── FlagProviders/            # IsLockedFlagProvider (adds lock flag to tree items)
│   ├── Interfaces/               # IContentLockService, IContentLockHubEvents
│   ├── Migrations/               # PackageMigrationPlan (DB table + admin permission)
│   ├── Models/                   # ContentLocks (DB model), ContentLockStatus/Overview
│   ├── Notifications/            # Handlers for ContentDeleting, ContentMovingToRecycleBin
│   ├── Options/                  # ContentLockOptions (IOptionsMonitor, appsettings)
│   ├── Services/                 # ContentLockService (core lock/unlock logic)
│   ├── SignalR/                  # ContentLockHub, ContentLockHubRoutes
│   └── wwwroot/App_Plugins/ContentLock/  # Compiled frontend output (do not edit manually)
├── ContentLock.Website/          # Umbraco test site for local development
├── ContentLock.E2E/              # E2E test project (.csproj)
├── ContentLock.Playwright.Tests/ # Playwright E2E tests (TypeScript)
├── build.ps1                     # PowerShell script to pack the NuGet package
├── appsettings-schema.umbraco.community.contentlock.json  # JSON schema for appsettings
└── Umbraco.Community.ContentLock.slnx  # Solution file
```

## Architecture

### Backend (C#)

- **`ContentLockService`** — Core service for lock/unlock/overview operations. Uses Umbraco's `IScopeProvider` / `NPoco` ORM to read/write the `ContentLocks` database table. Logs audit entries via `IAuditService`.
- **`ContentLockApiController`** — Versioned Umbraco backoffice API (`/umbraco/api/contentlock/v1/`). Endpoints: `GET Lock/{key}`, `GET Unlock/{key}`, `POST BulkUnlock`. After each operation, broadcasts the change via SignalR to all connected clients.
- **`ContentLockHub`** — SignalR hub (route: `/umbraco/ContentLockHub`). Tracks connected users with a `ConcurrentDictionary<Guid, ConcurrentHashSet<string>>` (one user can have multiple tabs/connections). On connect, sends the caller the current lock list, connected user list, and current options. Uses `IOptionsMonitor` to reactively push option changes to all clients.
- **`IsLockedFlagProvider`** — Implements `IFlagProvider` to attach a `Umbraco.ContentLock.Locked` flag to `DocumentTreeItemResponseModel`, `DocumentCollectionResponseModel`, and `DocumentItemResponseModel`. Used by the frontend to show visual lock indicators.
- **Notification Handlers** — `ContentDeletingNotificationHandler` and `ContentMovingToRecycleBinHandler` auto-unlock items when they are deleted or moved to the recycle bin (cancels the operation if a different user tries to delete a locked item).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [warrenbuckley/Umbraco.Community.ContentLock](https://github.com/warrenbuckley/Umbraco.Community.ContentLock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
