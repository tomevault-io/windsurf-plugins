---
trigger: always_on
description: Guidance for working on this codebase. Read this before making changes.
---

# AGENTS.md

Guidance for working on this codebase. Read this before making changes.

## Project Overview

Lyrictified Server is a local, Windows-first lyrics server for `.lrc`, `.elrc`, and `.ttml` files. It:

- Serves a public web UI (`/`, `/user`, `/submit`) for searching and downloading lyrics
- Scans a configured lyrics folder into an in-memory index on startup
- Persists admin-curated metadata in `data/catalog.json` (with a `.bak` backup)
- Accepts public lyric submissions for admin review, rate-limited per submitter
- Falls back to background LRCLIB.net searches when a local search is empty, caching results for admin review
- Provides a password-protected admin UI (`/admin`, `/admin/requests`, `/admin/lrclib-cache`)
- Runs a Windows system-tray icon with an "Open Admin"/"Exit" context menu
- Logs every request (method, path, status, duration, JSON response summary) to the console

## Tech Stack

- ASP.NET Core minimal hosting APIs (all endpoints defined inline in `Program.cs`, no controllers, no Razor)
- Target framework: `net10.0-windows`, `<UseWindowsForms>true</UseWindowsForms>` (required for the tray icon — this project is Windows-only)
- C# latest features: file-scoped namespaces, primary constructors/records, raw string literals (`"""`), `Lock` class for thread safety
- No third-party NuGet packages; only the BCL/ASP.NET Core framework
- All UI is static HTML + vanilla JS embedded as C# `const string` raw literals in the `*Page.cs` files

## Commands

```powershell
dotnet run                                   # build and run; default URL http://127.0.0.1:32145
dotnet run -- --hash-admin-password          # print a PBKDF2-SHA256 hash for AdminPasswordHash
dotnet build                                 # compile check
```

There are **no tests** in this repository. Verify changes with `dotnet build`, then manual smoke tests against the running server (e.g. `Invoke-RestMethod http://127.0.0.1:32145/health`).

## Configuration

`appsettings.json`, section `Lyrictified`:

| Key | Default | Purpose |
| --- | --- | --- |
| `Port` | `32145` | Listen port (validated 1–65535) |
| `BindAddress` | `127.0.0.1` | `0.0.0.0` exposes to LAN; `localhost` always means the same machine |
| `LyricsDirectory` | `lyrics` | Recursively scanned for `.lrc`/`.elrc`/`.ttml` |
| `CatalogPath` | `data/catalog.json` | Admin metadata; atomic save + `.bak` copy |
| `PendingSubmissionsPath` | `data/pending-submissions.json` | Submissions + 2h rate-limit records |
| `LrclibCachePath` | `data/lrclib-cache.json` | LRCLIB cache metadata |
| `LrclibCacheDirectory` | `lrclib-cache` | LRCLIB cached lyric files |
| `LrclibCacheAutoCleanMaxAgeDays` | `3` | Auto-clean age |
| `LrclibCacheAutoCleanCheckIntervalHours` | `1` | Auto-clean interval |
| `AdminPasswordHash` | `""` | Versioned PBKDF2-SHA256 hash; empty = admin disabled |
| `AdminPassword` | `""` | Legacy fallback (plaintext); logged as a warning |

Note: `Properties/launchSettings.json` and `Lyrictified.Server.http` still contain leftover template values (`localhost:5004`); the real port comes from `appsettings.json`. The `dotnet run` URL is decided by `builder.WebHost.UseUrls(...)` in `Program.cs:57`.

## Project Layout

All source `.cs` files live at the repository root (no `src/` folder).

| File | Responsibility |
| --- | --- |
| `Program.cs` | All HTTP endpoints, request-logging middleware, startup wiring, submission request parsing, password hash CLI |
| `Models.cs` | All records/models: settings, search request/result, lyric file, catalog entries, submissions, LRCLIB tracks |
| `LyricsIndex.cs` | Folder scan, catalog load/save, file-name metadata inference, search + weighted scoring, admin update |
| `PendingSubmissionStore.cs` | Submit/approve/reject flow, path sanitization, 2h rate limiting |
| `LrclibCacheStore.cs` | LRCLIB cached track CRUD, approve (copies into lyrics dir), reject, expiry cleanup |
| `LrclibSearchService.cs` | Background HTTP search against `https://lrclib.net/api/search` |
| `LrclibCacheAutoCleanService.cs` | `IHostedService` timer that prunes expired cache tracks |
| `TrayIconService.cs` | `IHostedService` STA thread running the Windows `NotifyIcon` |
| `AdminAuthService.cs` | Admin login sessions + `AdminPasswordHasher` (PBKDF2-SHA256) |
| `LyricOffsetHelper.cs` | Generates `.offset.ext` files applying a timestamp offset to LRC/TTML |
| `WeightedTagJsonConverter.cs` | Serializes `WeightedTag` as `{name, score}` or plain string |
| `UserPage.cs` / `SubmitPage.cs` / `AdminPage.cs` / `AdminRequestsPage.cs` / `AdminLrclibCachePage.cs` | Static HTML pages |

Data/runtime directories (not source): `lyrics/`, `data/`, `lrclib-cache/`. `bin/`, `obj/`, `.vs/`, `.vscode/` are gitignored.

## Key Conventions

- **Endpoints**: Add all routes in `Program.cs` with minimal API mapping. Admin routes live under `/admin/api/*` and must check `IsAdmin(context)` (validates the `lyrictified_admin` cookie via `AdminAuthService`) before acting, returning `Results.Unauthorized()` otherwise.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ios7jbpro/Lyrictified-Server](https://github.com/ios7jbpro/Lyrictified-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
