---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jellyfin plugin ("Jellyscribe") that syncs watch history to Letterboxd (film) and Serializd (TV). C#/.NET 9, targets Jellyfin 10.11 (`Jellyfin.Controller`/`Jellyfin.Model` 10.11.0). Letterboxd's official endpoint (`/api/v0/production-log-entries`) is preferred; the plugin falls back to web scraping (cookie login, CSRF tokens, HtmlAgilityPack) when the API path fails. Serializd's API needs no such fallback. The C# namespace, project folder, and solution file all still say `LetterboxdSync` (pre-rebrand name, unchanged this release, see `openspec/changes/rebrand-jellyscribe/`); only the compiled `AssemblyName` and every user-visible surface say Jellyscribe.

The sidebar link in the Jellyfin web UI depends on the third-party **File Transformation** plugin; the rest of the plugin works without it.

## Build & Test

```bash
dotnet build -c Release
dotnet test  -c Release --verbosity normal
```

Run a single test class or method (xUnit, `dotnet test` filter syntax):

```bash
dotnet test --filter "FullyQualifiedName~ScraperTests"
dotnet test --filter "FullyQualifiedName~ScraperTests.LookupBySlug_Returns_Result"
```

CI also collects coverage via `--collect:"XPlat Code Coverage"` into `TestResults/`; Codecov consumes the Cobertura XML.

Deploy a debug build to the local Jellyfin server: `./deploy.sh` (scp's `Jellyscribe.dll` + `HtmlAgilityPack.dll` and restarts the container).

## Architecture

### Service abstraction with fallback

`ILetterboxdService` (`ILetterboxdService.cs`) is the seam every caller uses. Two implementations:

- `LetterboxdApiClient`, preferred, talks to Letterboxd's JSON endpoints.
- `ScrapingLetterboxdService`, fallback, composes `LetterboxdHttpClient` (cookies/CSRF/Cloudflare retry), `LetterboxdAuth` (login + re-auth on 401), `LetterboxdScraper` (HTML parsing, film lookup, diary/watchlist scraping), and `LetterboxdDiary` (diary writes, review posting).

`LetterboxdServiceFactory.CreateAuthenticatedAsync` tries the API first and silently falls back to scraping if auth fails. The factory also exposes an `internal static OverrideForTesting` hook used via `InternalsVisibleTo` from `LetterboxdSync.Tests` to inject mock services, production code never touches it.

### Sync entry points

- `SyncTask`, scheduled, exports recent watches to the Letterboxd diary.
- `WatchlistSyncTask` / `WatchlistSyncRunner`, imports the user's Letterboxd watchlist as a Jellyfin playlist.
- `DiaryImportTask`, marks Jellyfin items as played if present in the Letterboxd diary.
- `PlaybackHandler`, `IHostedService` registered in `ServiceRegistrator`, fires the real-time sync on playback completion.
- `LetterboxdSyncRunner`, shared engine used by `SyncTask` and `PlaybackHandler`; `SyncGate`, `SyncHistory`, `SyncProgress`, and `TmdbCache` coordinate dedupe, progress UI, and TMDb lookups.

### Plugin surface

- `Plugin.cs` + `ServiceRegistrator.cs` register services and config.
- `Api/LetterboxdController.cs` and `Api/SidebarController.cs` expose REST endpoints consumed by the config dashboard. `LetterboxdController` also serves the read-only `ItemRating` endpoint the review modal uses to pre-fill its stars from the caller's stored Jellyfin rating.
- `Web/*.html` and `Web/*.js` are embedded resources (see `LetterboxdSync.csproj`) served as the plugin's config pages.
- `SidebarInjection.cs` registers a transformation with the File Transformation plugin to inject the sidebar link.

## Releasing

**Every merge to `main` that changes what ships, ships a release.** No manual tag pushes, no release-notes files. **We only release when a change affects the user; non-shipping PRs are exempt**: if the whole diff sits in non-shipping paths (any `*.md`, `docs/`, `openspec/`, `site/`, `worker/`, `.github/`, `LetterboxdSync.Tests/`), skip the version bump, the `## Release notes` section, and the `release-notes.ts` entry; the merge then ships no release (release.yml sees the existing tag and stops) and the site still redeploys for `site/**` changes via deploy-docs' push trigger. `manifest.json` is never exempt. The full pipeline is:

1. Open a PR. Unless non-shipping (above), the PR must:
   - Have a **Conventional Commits** title (`feat:`, `fix:`, `chore:`, `docs:`, `ci:`, `refactor:`, `test:`, `perf:`, `build:`, `style:`). Enforced by `pr-title.yml` (this one applies to non-shipping PRs too).
   - **Bump `AssemblyVersion` / `FileVersion`** in both `Directory.Build.props` and `LetterboxdSync/LetterboxdSync.csproj`. Patch bumps (e.g. `1.13.0.0` → `1.13.1.0`) are fine for CI / refactor changes. Enforced by `version-gate.yml`.
   - Fill in the **`## Release notes`** section in the PR body. `release.yml` extracts text between that heading and the next H2 and uses it verbatim as the manifest changelog field and the GitHub Release body. The PR template primes the section so it's the path of least resistance. Past entries on https://jellyscribe.dev/releases set the tone: one paragraph, user-facing prose, no symbol names / internal jargon.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [builtbyproxy/Jellyscribe](https://github.com/builtbyproxy/Jellyscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
