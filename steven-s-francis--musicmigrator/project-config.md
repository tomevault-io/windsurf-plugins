---
trigger: always_on
description: Validates state via `GetOAuthState`. Calls `SpotifyAuthHandler.ExchangeCodeAsync`. Stores the resulting `ProviderToken` in `ITokenStore`. Returns redirect to `http://localhost:5173/?connected=spotify`.
---


# MusicMigrator — Complete Implementation Blueprint
### For Antigravity 2.0 — Full Project from 0 to 100%


You are implementing the MusicMigrator project from scratch.
Work through the implementation sequence in Section 10 in order.
After each step, verify it builds or runs before proceeding.
Do not add anything not specified in this document.
---

## 1. PROJECT OVERVIEW

Build a web application called **MusicMigrator** that lets users migrate playlists and music libraries between three platforms: **Spotify**, **YouTube Music**, and **Anghami**.

The user authenticates with at least two platforms via OAuth, picks a source playlist, picks a destination platform, and the system matches and transfers every track. The UI shows live per-track progress.

---

## 2. TECHNOLOGY STACK

| Layer | Technology | Version |
|---|---|---|
| Backend runtime | .NET | 10 |
| Backend framework | ASP.NET Core Minimal API | 10 |
| Backend language | C# | 13 |
| Frontend framework | React | 19 |
| Frontend build tool | Vite | 6 |
| Frontend routing | react-router-dom | 7 |
| Frontend HTTP client | axios | latest |
| Spotify SDK | SpotifyAPI.Web NuGet | latest |
| YouTube SDK | Google.Apis.YouTube.v3 NuGet | latest |
| Browser automation | Microsoft.Playwright NuGet | latest |

No TypeScript. No test projects in this phase. No Docker.

---

## 3. REPOSITORY STRUCTURE

Single Git repository. Two top-level folders.

```
/ (repository root)
├── .gitignore
├── .git/
├── backend/
│   ├── MusicMigrator.sln
│   ├── MusicMigrator.Core/
│   ├── MusicMigrator.Providers/
│   └── MusicMigrator.API/
└── frontend/
    ├── package.json
    ├── vite.config.js
    ├── index.html
    └── src/
        ├── main.jsx
        ├── App.jsx
        ├── services/
        ├── pages/
        └── components/
```

---

## 4. GIT SETUP

### 4.1 Initialize

Run `git init` in the root. Set default branch to `main`.

### 4.2 .gitignore

The root `.gitignore` must cover both ecosystems:

**Ignore for .NET:**
`bin/`, `obj/`, `.vs/`, `*.user`, `*.suo`, `TestResults/`, `[Pp]ublish/`, `*.nupkg`, `appsettings.*.json` (but NOT `appsettings.json` itself), `**/secrets.json`

**Ignore for Node:**
`node_modules/`, `dist/`, `dist-ssr/`, `.env`, `.env.*` (but NOT `.env.example`), `.cache/`, `*.local`

**Ignore for OS:**
`.DS_Store`, `Thumbs.db`

**Ignore for Playwright:**
`/playwright-report/`, `/test-results/`

### 4.3 Final commit

After everything is implemented, built, and verified: stage all non-ignored files and commit with the message:
`Initial commit: MusicMigrator full implementation`

---

## 5. BACKEND ARCHITECTURE

### 5.1 Solution Structure

Three .NET 10 projects inside `/backend`:

| Project | Type | Purpose |
|---|---|---|
| `MusicMigrator.Core` | Class Library (`net10.0`) | Domain models, interfaces, and orchestration logic. No framework dependencies. |
| `MusicMigrator.Providers` | Class Library (`net10.0`) | Concrete implementations for Spotify, YouTube, and Anghami. |
| `MusicMigrator.API` | Web API (`net10.0`) | Minimal API host: endpoints, DI registration, middleware. |

**Project references:**
- `MusicMigrator.API` → references both `Core` and `Providers`
- `MusicMigrator.Providers` → references `Core` only
- `MusicMigrator.Core` → no project references

All three are added to `MusicMigrator.sln`.

---

### 5.2 NuGet Packages

**MusicMigrator.Core** requires:
- `Microsoft.Extensions.Logging.Abstractions` — for `ILogger<T>` in class libraries

**MusicMigrator.Providers** requires:
- `SpotifyAPI.Web` — official .NET Spotify client
- `Google.Apis.YouTube.v3` — official Google YouTube Data API v3 client
- `Microsoft.Playwright` — headless browser for Anghami write operations
- `Microsoft.Extensions.Configuration.Abstractions` — for `IConfiguration` injection
- `Microsoft.Extensions.Http` — for `HttpClient` typed clients

**MusicMigrator.API** requires no additional NuGet packages beyond what comes with the Web API template.

After `dotnet restore`, Playwright's Chromium browser must be installed before the app can run. See **Section 10 Step 11** for the correct platform-specific install command.

#### API Port Configuration

The API must run on port **5000** — this is what all OAuth redirect URIs and the Vite proxy target. After scaffolding `MusicMigrator.API`, open `MusicMigrator.API/Properties/launchSettings.json` and set the `applicationUrl` for the `http` profile to `http://localhost:5000`. Remove any HTTPS profile entries. The relevant section should look like:

```json
"MusicMigrator.API": {
  "commandName": "Project",
  "dotnetRunMessages": true,
  "launchBrowser": false,
  "applicationUrl": "http://localhost:5000",
  "environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
  }
}
```

Do not add `UseHttpsRedirection()` to `Program.cs`.

---

### 5.3 Domain Models (in `MusicMigrator.Core/Models/`)

Define all models in a single file or split by type. All models live in the `MusicMigrator.Core.Models` namespace.

**Playlist**
A record with: `Id` (string), `Name` (string), `Description` (string, nullable), `TrackCount` (int), `CoverUrl` (string, nullable).

**Track**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Steven-S-Francis/MusicMigrator](https://github.com/Steven-S-Francis/MusicMigrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
