---
trigger: always_on
description: FastMCP server providing sports data from The Odds API (betting odds) and ESPN API (games, teams, stats) for Claude Desktop. Includes optional web dashboard (React + Node.js) for bet tracking. Single Python server with dual data sources and rich formatting tools.
---

# BetTrack - AI Agent Instructions

## Project Overview
FastMCP server providing sports data from The Odds API (betting odds) and ESPN API (games, teams, stats) for Claude Desktop. Includes optional web dashboard (React + Node.js) for bet tracking. Single Python server with dual data sources and rich formatting tools.

## Project Structure
```
BetTrack/
├── mcp/                        # MCP server (Python FastMCP)
│   ├── sports_mcp_server.py    # Main server entry point
│   ├── sports_api/             # API handlers and formatters
│   ├── scripts/build/          # PowerShell build system
│   └── releases/               # Built MCPB packages
├── dashboard/                  # Optional web dashboard
│   ├── backend/                # Node.js + TypeScript + Prisma
│   └── frontend/               # React + Vite + Redux
└── docs/                       # Documentation
```

## Architecture Fundamentals

### Dual-API Design
- **MCP Server** (`mcp/sports_mcp_server.py`): FastMCP framework, stdio transport for Claude Desktop
- **Odds API Handler** (`mcp/sports_api/odds_api_handler.py`): Betting odds, scores, live lines
- **ESPN API Handler** (`mcp/sports_api/espn_api_handler.py`): Teams, scoreboards, standings, schedules
- **Formatters** (`mcp/sports_api/formatter.py`): Markdown tables, ASCII cards, visual scoreboards
- **Team Reference** (`mcp/sports_api/team_reference.py`): Team ID lookups, logo URLs for NFL/NBA/NHL

### Dashboard (Optional Web Component)
- **Backend** (`dashboard/backend/`): Node.js + Express + TypeScript + Prisma
  - **Routes**: games (timezone-aware filtering), bets, admin, mcp integration
  - **Services**: odds-sync (background), bet-service, outcome-resolver (background)
  - **Scheduled Jobs**: node-cron for automatic odds sync and outcome resolution
  - **API Features**: Background job execution, rate limit tracking, timezone handling
  - **Testing**: Jest 29+ with ts-jest, supertest for API testing
  - **Package**: @wford26/bettrack-backend (scoped npm package)
- **Frontend** (`dashboard/frontend/`): React + Vite + Redux Toolkit + Tailwind CSS
  - **Testing**: Vitest with React Testing Library, jsdom environment
  - **Build Tool**: Vite 6+ for fast HMR and optimized production builds
  - **Package**: @wford26/bettrack-frontend (scoped npm package)
- **Purpose**: Web UI for bet tracking, odds history, line movement charts
- **Database**: PostgreSQL 16-alpine with Prisma ORM 5.22.0
- **State Management**: Redux Toolkit with persistent storage
- **Docker**: Multi-service setup with postgres, backend, frontend, nginx

### Key Configuration
- **Single config source**: `.env` file (supports `.env.example` template)
- **Required**: `ODDS_API_KEY` (get from https://the-odds-api.com)
- **Optional**: `BOOKMAKERS_FILTER` (comma-separated, e.g., `draftkings,fanduel,betmgm`)
- **Optional**: `BOOKMAKERS_LIMIT` (default: 5 bookmakers per game)
- **Optional**: `LOG_LEVEL` (DEBUG, INFO, WARNING, ERROR, CRITICAL)

## Critical Development Workflows

### Pre-Push Build Verification

**IMPORTANT**: Always run build scripts before pushing changes to ensure production builds succeed.

**Build Script Location**: `scripts/build.ps1` (centralized build system)

```powershell
# Navigate to build script directorycd scripts

# Build MCP Server MCPB package only
.\build.ps1 -VersionBump patch  # or -Beta for testing

# Build Dashboard only (backend + frontend)
.\build.ps1 -Dashboard -BumpBackend -BumpFrontend

# Build everything (MCP + Dashboard)
.\build.ps1 -Dashboard -VersionBump patch -BumpBackend -BumpFrontend

# Beta build with version bumps
.\build.ps1 -Dashboard -BumpDashboard -BumpBackend -BumpFrontend -VersionBump patch -Beta

# Verify build outputs
ls ../../releases/  # Check for new .mcpb file
ls ../../../dashboard/dist/backend/  # Check for compiled JS files
ls ../../../dashboard/dist/frontend/  # Check for bundled assets
```

**Build verification checklist**:
- [ ] MCP server builds without errors (`scripts/build.ps1`)
- [ ] Backend TypeScript compiles (`npm run build` in dashboard/backend/)
- [ ] Frontend bundles successfully (`npm run build` in dashboard/frontend/)
- [ ] No TypeScript errors (`npm run type-check` if available)
- [ ] Tests pass (if applicable)
- [ ] Version numbers updated in manifest.json and package.json files

**When to build**:
- Before every commit that changes MCP server code
- Before every commit that changes dashboard backend/frontend
- After updating dependencies (package.json, requirements.txt)
- Before creating pull requests
- Before tagging releases

### Build System (PowerShell)
**Location**: `scripts/build.ps1` (centralized for MCP + Dashboard)

```powershell
# Navigate to build script
cd scripts

# Build MCP MCPB package only
.\build.ps1 -VersionBump patch

# Build Dashboard only (backend + frontend to dist/)
.\build.ps1 -Dashboard -BumpBackend -BumpFrontend

# Build everything (MCP + Dashboard)
.\build.ps1 -Dashboard -VersionBump patch -BumpBackend -BumpFrontend

# Beta build (uses git hash or incremental numbering)
.\build.ps1 -Beta  # MCP only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WFord26/BetTrack](https://github.com/WFord26/BetTrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
