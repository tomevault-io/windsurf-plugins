---
trigger: always_on
description: immich-lounge is a two-component system: a **companion service** (ASP.NET Core + Blazor Server) and a **Roku channel/screensaver** (BrightScript/BrighterScript). The companion handles configuration and playlist building; the Roku fetches media directly from Immich.
---

# AGENTS.md

## Project Overview

immich-lounge is a two-component system: a **companion service** (ASP.NET Core + Blazor Server) and a **Roku channel/screensaver** (BrightScript/BrighterScript). The companion handles configuration and playlist building; the Roku fetches media directly from Immich.

## Repo Conventions

- In user-facing copy, use the product name **Immich Lounge**. Keep lowercase `immich-lounge` only for technical slugs such as URLs, repo names, package names, Docker tags, image names, and protocol identifiers.
- The GitHub organization and GHCR owner are `immich-lounge`. Public docs should use `https://github.com/immich-lounge/immich-lounge` and `ghcr.io/immich-lounge/immich-lounge-companion`.
- The companion defaults to port `4383` for local dev, Docker, and manual Roku setup.
- Keep public docs and README copy concise and plain. Avoid hype, filler, and over-marketing language.
- Public docs use **Zensical** with the existing `mkdocs.yml` compatibility path, not Jekyll. Main public routes are `/support`, `/privacy`, and `/tos`.
- Public website content lives under `docs/website/`. Reviewer and maintainer docs live under `docs/reviewer-guide/` and `docs/maintainer/`.
- Public docs should prefer the website URL `https://immich-lounge.github.io/` when linking users outward.
- Keep the README product-first. Public setup and product documentation should usually live in `docs/website/` or `CONTRIBUTING.md`, not in the README.
- When a change affects user-visible behavior, setup, settings, or other public-facing workflows, update the relevant docs in the same change unless the deferral is explicit.
- Public setup docs should describe the manual companion URL flow. Do not describe SSDP/discovery as a supported public setup path, and do not recommend Docker host networking in public docs.
- The Roku code uses split registry keys for channel and screensaver state. Do not reintroduce old public docs that describe a single `profileId` / `cachedProfile` / `cachedPlaylist` model.
- The companion Docker runtime image should use the chiseled extra ASP.NET image.
- For Roku quality checks, use Roku's official Static Analysis Tool (`roku/scripts/run-static-analysis.ps1`, `npm run analyze*`) plus `@rokucommunity/bslint`. The official tool is the certification-facing check; `bslint` is the fast local/CI preflight linter.
- Keep a technical changelog in `CHANGELOG.md` and a user-facing release summary in `docs/website/changelog.md`. When one is updated for a release, keep the other in sync and link between them.
- Put non-source local build output under `_out/` whenever possible. Website builds go to `_out/website/`; ad hoc local or agent-generated output should prefer a dedicated subfolder such as `_out/codex/`.
- Treat `companion/src/ImmichLoungeCompanion/artifacts/`, `tools/**/bin/`, `tools/**/obj/`, and generated build output as disposable and not source.
- Keep local planning/spec work under `.planning/`, not under `docs/`.
- `.planning/` is local-only and gitignored. Do not commit it or move its contents back into the public docs unless explicitly asked.
- Treat untracked local tooling folders such as `.claude/`, `.superpowers/`, `.playwright-mcp/`, and `tools/` carefully. Do not commit or clean them up unless explicitly requested.
- Branding SVGs live in `branding/`. To regenerate PNGs and deploy them to `roku/images/`, `docs/website/assets/`, and the companion wwwroot, run `node branding/render-branding.mjs`. This requires **Inkscape** (`C:\Program Files\Inkscape\bin\inkscape.exe`). Do not use other SVG-to-PNG tools (cairosvg, resvg, etc.) — they produce poor output for these assets. After running, commit the updated PNGs alongside any SVG changes.

---

## Companion Service (`companion/`)

**Tech stack:** .NET 10, ASP.NET Core, Blazor Server, MSTest, NSubstitute

### Commands (run from `companion/`)

```bash
# Run locally (dev port 4383)
dotnet run --project src/ImmichLoungeCompanion

# Run all tests
dotnet test --project tests/ImmichLoungeCompanion.Tests/ImmichLoungeCompanion.Tests.csproj

# Run a single test class or method
dotnet test --project tests/ImmichLoungeCompanion.Tests/ImmichLoungeCompanion.Tests.csproj --filter "FullyQualifiedName~PlaylistCache"

# Build Docker image
docker build -t immich-lounge-companion .

# Deploy
docker-compose up -d
```

### Key Architecture

- **`Api/`** — MVC controllers (`ProfilesController`, `PlaylistController`, `SettingsController`, `ImmichController`) expose the REST API consumed by the Roku
- **`Components/Pages/`** — Blazor Server pages: `Connection.razor`, `Profiles.razor`, `ProfileEditor.razor`
- **`Playlist/`** — `PlaylistCacheWorker` (hosted service), `PlaylistCache`, `PlaylistBuilder`; cache lifecycle: cold → `{ building: true }`, warm → served immediately, proactive rebuild 20% before expiry
- **`Storage/`** — `JsonSettingsRepository` and `JsonProfileRepository` write to `/data/settings.json` and `/data/profiles/<id>.json`
- **`Immich/`** — `ImmichClient` calls Immich REST API for search and memories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [immich-lounge/immich-lounge](https://github.com/immich-lounge/immich-lounge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
