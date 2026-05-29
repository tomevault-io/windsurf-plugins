---
trigger: always_on
description: - Never commit changes unless explicitly asked to do so.
---

# Copilot Instructions

## General Rules

- Never commit changes unless explicitly asked to do so.
- **Never deploy to production without explicit user confirmation.** Before invoking the `deploy` skill or running any `swa deploy`, `docker push`, or `az containerapp` update command, always stop and ask the user: "Ready to deploy to production?" and wait for a clear yes.
- **Always validate UI changes locally before marking a task complete.** After any change to Razor components, CSS, or JavaScript, start the local stack (use the `run-locally` skill) and use Playwright to verify the behavior in the browser. Do not consider a UI task done until it has been visually confirmed working locally.

## Architecture

Three-tier application hosted on Azure:

```
Browser → Azure Static Web Apps (Easy Auth)
              → /api/* → Azure Functions proxy (api/)
                            → ASP.NET Core backend (src/Server/)
                                  → SQLite database
```

- **`src/WasmApp/`** — Blazor WebAssembly frontend (C#, Razor, Bootstrap)
- **`src/Server/`** — ASP.NET Core Web API backend with SQLite
- **`src/Shared/`** — Shared DTOs/contracts used by both frontend and backend
- **`api/`** — Azure Functions v4 (Node.js) API proxy
- **`test/SerializerTests/`** — MSTest unit tests (net9.0)
- **`infrastructure/`** — Bicep templates for Azure Container Apps + SWA

## Build & Test

```powershell
# Build the whole solution
dotnet build rss-reader.sln

# Run all tests
dotnet test rss-reader.sln

# Run a single test class
dotnet test test/SerializerTests --filter "ClassName=SerializerTests.SerializerTests"

# Build Docker image for the backend (run from repo root)
docker build -f src/Server/Dockerfile -t rss-reader-api:local .

# Build and deploy the frontend (run from rss-reader/)
swa build
swa deploy --env production
```

**Local dev**: Hit F5 in Visual Studio — launches frontend and backend together.

For containerized backend testing, see [LOCAL-TESTING.md](../LOCAL-TESTING.md).

## ⛔ Local Dev Build Rule: Always Debug, Never Release

**When running locally, ALWAYS use `dotnet build -c Debug`. NEVER run `dotnet publish -c release` or `dotnet build -c Release` for local testing.**

The release build excludes `appsettings.Development.json` from the output (`CopyToPublishDirectory = Never`). This file contains `EnableTestAuth: true`, which is what allows the test user (`testuser2`) to bypass SWA Easy Auth locally. Without it, every API call returns 401 and the app appears broken even though the code is correct.

The `swa start rss-reader-local` config uses `dotnet watch run` (Kestrel dev server on port 8443) as the app source — it serves files directly from the source `wwwroot/`, not from a publish output directory. The `outputLocation` in `swa-cli.config.json` for the `rss-reader-local` config points to `bin/debug/net9.0/wwwroot` (the debug build output). Never use a release build or create a release placeholder directory for local testing.

## Authentication Flow

Authentication is a two-stage chain:

1. **Azure SWA Easy Auth** authenticates the browser via AAD (login at `/.auth/login/aad`). SWA injects `x-ms-client-principal` (base64 JSON) into requests to the Function proxy — browsers cannot forge this.

2. **Azure Functions proxy** (`api/src/functions/ApiProxy.js`) extracts the Easy Auth principal, then forwards requests to the backend with:
   - `X-Gateway-Key`: a shared secret (env var `RSSREADER_API_KEY`)
   - `X-User-Id`: the raw base64 `x-ms-client-principal` value

3. **Backend** (`StaticWebAppsAuthenticationHandler`) validates `X-Gateway-Key` against `RSSREADER_API_KEY` env var, then parses `X-User-Id` to build the `ClaimsPrincipal`.

For local development, set `RssAppConfig__IsTestUserEnabled=true` to bypass auth with a fake test user (`testuser2`).

## Key Conventions

### Shared Contracts
All DTOs live in `src/Shared/Contracts/`: `NewsFeed`, `NewsFeedItem`, `RssUser`, `OpmlImport`. Both the frontend (`FeedClient`/`UserClient`) and backend controllers use these same types.

### Repository Pattern
The backend uses interfaces (`IFeedRepository`, `IItemRepository`, `IUserRepository`) with SQLite implementations in `src/Server/Data/`. Repositories are registered as singletons and initialize their own database tables on construction. The order of singleton creation matters (see `Program.cs` — feed → user → item).

### Background Work
Feed refresh runs via a `BackgroundWorkQueue` + `BackgroundWorker` hosted service. Enqueue work items to `BackgroundWorkQueue`; `FeedRefresher` handles the actual HTTP fetch and parse. Don't call `FeedRefresher` directly from controllers — enqueue instead.

### Configuration
App config is loaded into `RssAppConfig` from `appsettings.json` under the `RssAppConfig` section. The backend reads `DbLocation` for the SQLite path (`/tmp/storage.db` in Docker, copied to `/data/` for persistence). Frontend config is in `RssWasmConfig`.

### DatabaseBackupService

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brandonchastain/rss-reader](https://github.com/brandonchastain/rss-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
