---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and other AI coding agents when working with code in this repository. It is the single source of truth for project conventions; `AGENTS.md` is a symlink to this file so all tools read the same guidance.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and other AI coding agents when working with code in this repository. It is the single source of truth for project conventions; `AGENTS.md` is a symlink to this file so all tools read the same guidance.

## This is a public, open-source repository

This repo is public on GitHub under the MIT license. Treat everything you commit as world-readable, permanent, and indexed.

- **Never commit secrets or credentials.** API keys, OAuth client secrets, Postgres passwords, Resend keys, etc. always come from environment variables, AppHost parameters, or user-secrets — never from tracked files. The committed `appsettings*.json` keep these fields empty; keep them that way.
- **No private/personal data.** No personal emails, internal hostnames, IP addresses, private URLs, server names, or deployment endpoints in tracked files. Deployment targets (Dokploy URL/key, etc.) live only in GitHub Actions secrets.
- **No local planning artifacts.** Don't commit scratch design docs, plan files, transcripts, or `.claude/` decision notes — those belong in your local environment, not the public history.
- If you're unsure whether something is safe to publish, leave it out and ask.

## Commands

All commands run from the repo root unless noted.

```bash
# Run the full stack (Aspire dashboard at https://localhost:17072)
# Provisions PostgreSQL in Docker, starts API + frontend, auto-applies EF migrations.
dotnet run --project MusicHoarder.AppHost

# First run: required values are modeled as AppHost parameters and prompted in the
# dashboard. To pre-seed (recommended for repeatable boots) set them as AppHost
# user-secrets — note the `Parameters:` prefix and the AppHost project:
dotnet user-secrets set "Parameters:source-directory" "/tmp/musichoarder-source" --project MusicHoarder.AppHost
dotnet user-secrets set "Parameters:destination-directory" "/tmp/musichoarder-dest" --project MusicHoarder.AppHost
# Optional (otherwise dashboard prompts as blank, providers gracefully degrade):
dotnet user-secrets set "Parameters:acoustid-api-key" "..." --project MusicHoarder.AppHost
dotnet user-secrets set "Parameters:spotify-client-id" "..." --project MusicHoarder.AppHost
dotnet user-secrets set "Parameters:spotify-client-secret" "..." --project MusicHoarder.AppHost

# Tests (xUnit, in-memory EF provider — no Postgres/Docker required)
dotnet test MusicHoarder.Api.Tests/MusicHoarder.Api.Tests.csproj

# Run a single test class / test
dotnet test MusicHoarder.Api.Tests/MusicHoarder.Api.Tests.csproj --filter "FullyQualifiedName~EnrichmentOrchestratorTests"
dotnet test MusicHoarder.Api.Tests/MusicHoarder.Api.Tests.csproj --filter "DisplayName~matches_song_via_acoustid"

# Frontend standalone (point at API port from the Aspire dashboard)
cd frontend && MUSICHOARDER_API_URL=http://localhost:<api-port> PORT=3000 bun run dev
cd frontend && bun run build        # SvelteKit + adapter-node build
cd frontend && bun run check        # svelte-check + TypeScript
cd frontend && bun run lint         # ESLint (flat config)
```

CI (`.github/workflows/ci.yml`) builds and tests `MusicHoarder.Api.Tests` and the frontend; the Android client has its own path-filtered `android.yml` (unit tests + both APK variants, only when `android/**` changes). Only `dotnet` and `frontend` are required status checks on `main` — which is what makes the Android path filter safe. A separate `release.yml` runs unified semantic-release (API + frontend) on every push to `main` — see **Releases** below. Docker must be running locally before `AppHost` starts, because Aspire provisions PostgreSQL as a container.

## Solution layout

- **`MusicHoarder.Api`** — ASP.NET Core minimal API. Composition root is `Program.cs` → `AddMusicHoarderServices()` + `MapMusicHoarderEndpoints()`. Hosts the full pipeline as `BackgroundService`s and EF Core persistence (Npgsql).
- **`MusicHoarder.AppHost`** — Aspire entry point. Wires Postgres (`ContainerLifetime.Persistent` + named data volume), API, and the SvelteKit frontend (`AddViteApp(...).WithBun()` with an HTTPS endpoint and Aspire dev cert). All required secrets/paths are modeled as `AddParameter(...)` and injected into the API as env vars (`MusicEnricher__*`, `Spotify__*`); the dashboard prompts for any missing values on first run. Frontend gets `MUSICHOARDER_API_URL` (HTTP for the internal Node→ASP.NET proxy hop); API gets `Frontend__PublicBaseUrl` (HTTPS, this env's own origin — the Spotify relay bounces the browser back here). Spotify OAuth uses one registered relay URI shared by every env (`Spotify__OAuthRelayUrl`); the relay route lives on the frontend (`/api/spotify/relay`) and is gated by a shared HMAC-signed `state` (`Spotify__OAuthStateSigningKey` / frontend `SPOTIFY_OAUTH_STATE_SIGNING_KEY`) plus a return-origin allowlist (`SPOTIFY_RETURN_ORIGIN_ALLOWLIST`) — see README "Spotify OAuth (relay)". `AddDockerComposeEnvironment("compose")` lets `aspire publish` emit a `docker-compose.yml` for Dokploy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jeffreyyvdb/MusicHoarder](https://github.com/Jeffreyyvdb/MusicHoarder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
