---
trigger: always_on
description: This repo hosts a .NET 9 Blazor WebAssembly app for a board‑game scorekeeper. Agents should prioritize end‑to‑end edits, verify with a local build, and keep UI consistent with the existing glass/white theme.
---

# AI Agent Quickstart for MyScoreBoard

This repo hosts a .NET 9 Blazor WebAssembly app for a board‑game scorekeeper. Agents should prioritize end‑to‑end edits, verify with a local build, and keep UI consistent with the existing glass/white theme.

## Architecture and data flow
- UI: Razor pages/components in `MyScoreBoard/Pages`, layout in `MyScoreBoard/Layout`, small UI components in `MyScoreBoard/Components`.
- State: `Services/GameService.cs` is the single source of truth for the current game (players, rounds, totals). It also orchestrates persistence.
- Persistence:
  - IndexedDB via JS interop (`wwwroot/js/indexedDb.js`) wrapped by `Services/IndexedDbService.cs`.
  - Active game cached in the `active` store (fixed key `current`), completed games in `games` (auto‑increment keys). DB versioning is in `indexedDb.js`.
  - `LocalStorageService.cs` mirrors a quick boolean `hasActiveGame` for fast UI (Home) and preferences (saved setup values).
- Celebrations/UI polish: `Components/Confetti.razor` and `wwwroot/js/confetti.js`.

```instructions
# AI Agent Quickstart for MyScoreBoard (Blazor + MAUI)

This monorepo contains a Blazor WebAssembly web app (`MyScoreBoard`), a MAUI host (`MyScoreBoardMaui`) and a shared library (`MyScoreBoardShared`) with models and service interfaces. Agents should prefer end-to-end edits, use DI interfaces from `MyScoreBoardShared`, and verify via local builds (web and MAUI Android for CI-free verification).

Architecture & big picture
- Projects: `MyScoreBoard` (Blazor), `MyScoreBoardMaui` (MAUI), `MyScoreBoardShared` (models + interfaces).
- Shared types: `MyScoreBoardShared/Models` holds `Player`, `Round`, `GameSession`, `GameStoreEntry`.
- Service interfaces: `MyScoreBoardShared/Services` includes `IGameService`, `IIndexedDbService`, `ILocalStorageService` — prefer these for edits and DI wiring.
- Data flow: `GameService` (web) is the app state orchestrator; it calls `IIndexedDbService` + `ILocalStorageService` for persistence.

Persistence backends (important)
- Web: `wwwroot/js/indexedDb.js` + `MyScoreBoard/Services/IndexedDbService.cs` (JS interop). Stores: `active` (explicit key `current`) and `games` (auto-increment).
- MAUI: sqlite-backed `IIndexedDbService` implementation (uses `sqlite-net-pcl` and `SQLiteAsyncConnection`) at `MyScoreBoardMaui/Services/IndexedDbService.cs` and `ILocalStorageService` implemented using `Preferences` at `MyScoreBoardMaui/Services/LocalStorageService.cs`.

Dependency injection & lifetimes
- Web DI registrations live in `MyScoreBoard/Program.cs` (scoped): bind `IGameService`, `IIndexedDbService`, `ILocalStorageService` to Blazor implementations.
- MAUI DI registrations live in `MyScoreBoardMaui/MauiProgram.cs` (singleton for platform services): bind shared interfaces to MAUI implementations.
- When editing services, update both registration locations and prefer constructor injection of the shared interfaces.

CSS architecture & styling (important)
- **Shared CSS**: `MyScoreBoardShared/wwwroot/css/app.css` (617 lines) contains the complete component foundation - CSS variables, glass effects, animations, all shared component styles. This is the single source of truth for styling.
- **Web CSS**: `MyScoreBoard/wwwroot/css/app.css` (321 lines) contains web-specific enhancements only - base layout (html/body background), Blazor loading UI, web-specific overrides for better contrast.
- **MAUI CSS**: `MyScoreBoardMaui/wwwroot/css/app.css` (374 lines) contains mobile-specific optimizations - touch targets (44px minimum), safe area handling, iOS/Android platform tweaks, mobile layout adjustments.
- **Static Assets**: All Bootstrap CSS/JS and Bootstrap Icons are centralized in `MyScoreBoardShared/wwwroot/` and referenced via `_content/MyScoreBoardShared/` path in both hosts.
- **CSS Loading Order**: Both apps load in this order: Bootstrap CSS → Platform CSS → Shared CSS → Generated styles. This ensures proper cascade and override behavior.
- **When adding new components**: Add base styles to shared CSS first, then platform-specific tweaks to web/MAUI CSS if needed. Always test both platforms after changes.
- **Bootstrap Integration**: Local Bootstrap Icons (no CDN), centralized in shared project. Icon usage: `bi-*` classes work across all platforms. Bootstrap components: cards, buttons, forms, modals, navbar, badges, alerts, grid all supported.

Build / dev workflow (practical)
- Web: cd `MyScoreBoard` → `dotnet build` / `dotnet run` (serves the Blazor app).
- MAUI: prefer Android target for local CI-free checks: `dotnet build MyScoreBoardMaui/MyScoreBoardMaui.csproj -f net9.0-android`.
- Full solution: `dotnet build MyScoreBoard.sln` (note: iOS/MacCatalyst builds may fail locally if Xcode version mismatch; see note below).

Key files to inspect for edits
- UI: `MyScoreBoard/Pages/*` (`Home.razor`, `GameSetup.razor`, `GamePlay.razor`, `GameHistory.razor`) - now located in `MyScoreBoardShared/Pages/*` for cross-platform sharing.
- Layout: `MyScoreBoardShared/Layout/*` (`MainLayout.razor`, `NavMenu.razor`) - shared layout components with glass card theme.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesmontemagno/blazor-scorekeeper](https://github.com/jamesmontemagno/blazor-scorekeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
