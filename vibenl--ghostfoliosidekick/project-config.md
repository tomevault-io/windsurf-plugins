---
trigger: always_on
description: **GhostfolioSidekick**: .NET 10.0 sidecar for [Ghostfolio](https://github.com/ghostfolio/ghostfolio) (wealth mgmt platform). Contains:
---

# GhostfolioSidekick - Copilot Instructions

## Repository Overview

**GhostfolioSidekick**: .NET 10.0 sidecar for [Ghostfolio](https://github.com/ghostfolio/ghostfolio) (wealth mgmt platform). Contains:

1. **GhostfolioSidekick** - Docker container, auto-imports broker/crypto transactions to Ghostfolio hourly
2. **PortfolioViewer** - Blazor WebAssembly + .NET Aspire for portfolio analysis/visualization
3. **AI Components** - Semantic Kernel + Web LLM for chat
4. **Supporting Libraries** - Parsers, DB, API integrations, utilities

**Size**: ~45 projects, single solution  
**Lang**: C# 14.0 (preview)  
**Framework**: .NET 10.0 (preview)  
**Tech**: Blazor WebAssembly, .NET Aspire, EF Core, Playwright (UI testing), xUnit

## Build & Validation

### Prerequisites

- **.NET 10 SDK** (10.0.102+)
- **WASM workload**: `dotnet workload install wasm-tools` before first build
- **Playwright**: Browsers install during test runs
- **Mono** (Linux CI only): Some build tasks
- **Node.js**: TypeScript + Playwright

### Build Commands

Build all 45 projects:
dotnet build
**Expected**: Succeeds ~15s, up to 2 warnings OK. Warnings = errors via `Directory.Build.props` `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`.

**Important**: Run `dotnet workload restore` after clone or if WASM builds fail.

### Testing

**Run all tests**:
dotnet test
**Tests with coverage** (CI):
dotnet tool install --global dotnet-coverage
dotnet-coverage collect "dotnet test" -f xml -o "coverage.xml"
**Install Playwright browsers** (before UI tests):
# Install for all test projects
find . -type f -name 'playwright.ps1' | while read script; do
  pwsh "$script" install
done

# Or on Windows PowerShell:
Get-ChildItem -Recurse -Include "playwright.ps1" | ForEach-Object { pwsh $_.FullName install }
**Test Projects**: `*.UnitTests` suffix + integration tests:
- `PortfolioViewer.WASM.UITests` - Playwright UI tests (screenshots/videos on failure)
- `IntegrationTests` - General integration tests

### Running the Application

**Development (Aspire AppHost)**:
dotnet run --project PortfolioViewer/PortfolioViewer.AppHost/PortfolioViewer.AppHost.csproj
Launches Aspire dashboard + API service + Blazor WASM client.

**GhostfolioSidekick Console App**:
dotnet run --project GhostfolioSidekick/GhostfolioSidekick.csproj
**Docker Build** (see `Dockerfile`):
- Multi-stage build
- Installs Python, Node.js, wasm-tools, supervisord
- Builds PortfolioViewer.ApiService, PortfolioViewer.WASM, GhostfolioSidekick
- Platform-specific via `TARGETARCH`

## Project Layout & Architecture

### Repository Structure
/
├── .github/workflows/docker-publish.yml    # Main CI/CD pipeline
├── Directory.Build.props                   # TreatWarningsAsErrors=true for entire solution
├── .editorconfig                          # Code style: tabs, CRLF, charset=utf-8
├── Dockerfile                             # Multi-stage build for production
├── GhostfolioSidekick.slnx               # Solution file (XML format)
│
├── GhostfolioSidekick/                   # Main console application (entry point)
├── PortfolioViewer/                      # Blazor WASM viewer application
│   ├── PortfolioViewer.AppHost/         # .NET Aspire orchestration (Program.cs is entry point)
│   ├── PortfolioViewer.WASM/            # Blazor WebAssembly client (uses Aspire4Wasm)
│   ├── PortfolioViewer.WASM.Data/       # Data services, models, and EF Core logic
│   ├── PortfolioViewer.ApiService/      # Backend API service
│   ├── PortfolioViewer.Common/          # Shared types
│   ├── PortfolioViewer.ServiceDefaults/ # Aspire service defaults
│   ├── PortfolioViewer.WASM.AI/         # AI integration (WebLLM chat)
│   └── *UnitTests/                      # Test projects (xUnit)
│
├── AI/                                   # AI components using Semantic Kernel
│   ├── AI.Agents/                       # AI agent implementations
│   ├── AI.Functions/                    # AI function definitions
│   ├── AI.Server/                       # AI server component
│   └── AI.Common/                       # Shared AI logic
│
├── Database/                            # Entity Framework Core with SQLite
├── GhostfolioAPI/                       # Client for Ghostfolio REST API
├── Parsers/                             # Transaction parsers for various brokers
├── Model/                               # Domain models
├── Configuration/                       # Configuration management (JSON-based)
├── ExternalDataProvider/                # Yahoo, CoinGecko, DividendMax integrations
├── PerformanceCalculations/             # Portfolio performance calculations
├── Cryptocurrency/                      # Crypto-specific logic
├── Utilities/                           # Common utilities
└── Tools/                               # Utility tools (AnonymisePDF, ScraperUtilities)

### Key Architectural Patterns

1. **Blazor WebAssembly + .NET Aspire**: PortfolioViewer use Aspire for orchestration (`PortfolioViewer.AppHost`). WASM client ↔ `PortfolioViewer.ApiService` backend.

2. **EF Core + SQLite**: All DB ops via `Database/DatabaseContext`. Migrations in `Database/Migrations/` (excluded from SonarQube).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VibeNL/GhostfolioSidekick](https://github.com/VibeNL/GhostfolioSidekick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
