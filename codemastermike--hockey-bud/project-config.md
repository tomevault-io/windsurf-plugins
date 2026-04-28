---
trigger: always_on
description: Auto-generated from all feature plans. Last updated: 2026-04-11
---

# hockey-site Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-04-11

## Active Technologies
- C# 14 / .NET 10 (backend), TypeScript 5.x / Angular 19 (frontend) + ASP.NET Core 10, Entity Framework Core 10, Hangfire, SignalR, Angular SSR, RxJS, Tailwind CSS v3, Angular CDK (001-hockey-league-hub)
- SQL Server (Azure SQL Database Serverless for deployed, SQL Server 2022 Developer for local Docker), Redis 7 (cache + pub/sub for real-time) (001-hockey-league-hub)

## Project Structure

```text
backend/
├── src/
│   ├── HockeyHub.Core/             # Entities + interfaces (no dependencies)
│   │   ├── Models/Entities/         # League, Team, Season, Arena, Player, Personnel, FranchiseHistory, Game, GamePeriodScore, StandingsSnapshot, Trade, TradeAsset
│   │   ├── Providers/               # INhlDataProvider interface + DTOs, IScoreBroadcaster
│   │   └── NhlDateHelper.cs         # Shared NHL game day boundary logic (3 AM ET cutoff, DST-aware)
│   ├── HockeyHub.Data/             # Data access (depends on Core)
│   │   ├── Data/HockeyHubDbContext.cs + Migrations/
│   │   ├── Providers/NhlWebApiProvider.cs
│   │   └── Services/
│   │       ├── Cache/               # Redis cache service
│   │       ├── Sync/                # DataSeed, ScoresSync, StandingsSync, ScheduleSync, TradeSync jobs
│   │       └── Queries/             # Scores, Standings, Schedule, Teams, GameHub, Trades, SeasonMode, PlayoffBracket, Draft query services
│   └── HockeyHub.Api/              # HTTP host (depends on Data + Core)
│       ├── Controllers/             # Scores (5), Standings, Schedule, Teams (2), GameHub, Trades, Search, Health, SeasonMode, PlayoffBracket, Draft
│       ├── Hubs/                    # GameHub, SignalRScoreBroadcaster
│       ├── Middleware/              # Error handling, DataAsOf wrapper
│       ├── Program.cs              # App startup + DI wiring
│       └── appsettings.json        # Connection strings
├── Dockerfile                       # Multi-stage build (SDK → runtime)
├── .dockerignore
└── tests/HockeyHub.Api.Tests/

infra/
├── main.bicep                       # All Azure resources (Container Apps, ACR, Key Vault, App Insights, etc.)
├── parameters.dev.json              # Dev environment parameters
└── parameters.prod.json             # Prod environment parameters

.github/workflows/
├── ci.yml                           # PR gate: build, test, lint (both backend + frontend)
├── deploy-dev.yml                   # On merge to main: build → push ACR → migrate → deploy → smoke test
└── deploy-prod.yml                  # Manual/release: same with approval gate

frontend/
├── src/app/
│   ├── components/
│   │   ├── layout/                # Banner, NavBar, ScoreBar (live), HamburgerMenu
│   │   ├── shared/                # StatTable, VideoModal, Pagination
│   │   ├── main-page/             # League selection grid
│   │   ├── scores/                # ScoresPage, ScoreBox, ExpandedScoreBox, PregameMatchup, CalendarPicker
│   │   ├── standings/             # StandingsPage (4 views: wildcard/division/conference/league)
│   │   ├── schedule/              # SchedulePage (monthly game list with navigation)
│   │   ├── teams/                 # TeamsIndex (card grid) + TeamProfile (roster table)
│   │   ├── game-hub/              # GameHubPage (team stats, player stats, goals/penalties)
│   │   ├── trades/                # TradesList (chronological trade cards)
│   │   ├── playoffs/              # PlayoffBracketPage (conference tabs, matchup cards), MatchupDetailPage
│   │   ├── draft/                 # DraftPage (round tabs, pick table with clickable player links)
│   │   └── [stats,...]/           # Placeholder route components (5 remaining)
│   ├── constants.ts               # Shared constants (league ID, polling intervals, SignalR config, close-game thresholds, getPeriodLabel)
│   ├── services/                  # Theme, SignalR, ScoresApi, StandingsApi, ScheduleApi, SearchApi, TeamsApi, GameHubApi, TradesApi, GameClock, SeasonMode, PlayoffsApi, DraftApi
│   ├── directives/                # TooltipDirective
│   ├── pipes/                     # EraPipe, TimezonePipe
│   ├── app.routes.ts              # 18 lazy-loaded routes (incl. game-hub/:gameId, teams/:teamId, playoffs, playoffs/matchup/:seriesLetter, draft)
│   └── app.ts                     # Shell: banner + score bar + nav + router-outlet
├── src/assets/fonts/              # Courier Prime (WOFF2)
├── src/styles/                    # Design tokens (light/dark)
├── staticwebapp.config.json         # Azure Static Web Apps routing + headers
├── e2e/                             # Playwright e2e tests
│   ├── tests/                       # 7 spec files (navigation, standings, teams, search, schedule, trades, game-hub)
│   └── pages/                       # Page Object Models (standings, teams, search)
├── playwright.config.ts             # Playwright config (desktop + mobile projects, auto-start ng serve)
└── tests/
```

## Commands

```bash
# Backend
cd backend && dotnet build                            # Build
cd backend && dotnet test                             # Tests
cd backend/src/HockeyHub.Api && dotnet run            # Run API (http://localhost:5072)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CodeMasterMike) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
