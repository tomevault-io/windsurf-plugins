---
trigger: always_on
description: Monorepo for Umbraco.AI and add-on packages. Each product has its own `.slnx`, `CLAUDE.md`, and builds independently.
---

# CLAUDE.md

## Repository Structure

Monorepo for Umbraco.AI and add-on packages. Each product has its own `.slnx`, `CLAUDE.md`, and builds independently.

| Product | Location | Category |
|---------|----------|----------|
| Umbraco.AI | `Umbraco.AI/` | Core |
| Umbraco.AI.Agent | `Umbraco.AI.Agent/` | Add-on |
| Umbraco.AI.Agent.UI | `Umbraco.AI.Agent.UI/` | Add-on (chat UI library) |
| Umbraco.AI.Agent.Copilot | `Umbraco.AI.Agent.Copilot/` | Add-on (frontend-only) |
| Umbraco.AI.Prompt | `Umbraco.AI.Prompt/` | Add-on |
| Umbraco.AI.Search | `Umbraco.AI.Search/` | Add-on |
| Umbraco.AI.Deploy | `Umbraco.AI.Deploy/` | Deploy |
| Umbraco.AI.Prompt.Deploy | `Umbraco.AI.Prompt.Deploy/` | Deploy |
| Umbraco.AI.Agent.Deploy | `Umbraco.AI.Agent.Deploy/` | Deploy |
| Umbraco.AI.Automate | `Umbraco.AI.Automate/` | Automate |
| Umbraco.AI.OpenAI | `Umbraco.AI.OpenAI/` | Provider |
| Umbraco.AI.Anthropic | `Umbraco.AI.Anthropic/` | Provider |
| Umbraco.AI.Amazon | `Umbraco.AI.Amazon/` | Provider |
| Umbraco.AI.Google | `Umbraco.AI.Google/` | Provider |
| Umbraco.AI.MicrosoftFoundry | `Umbraco.AI.MicrosoftFoundry/` | Provider |

### Dependency Tree

```
Umbraco.AI (Core)
├── Providers: OpenAI, Anthropic, Amazon, Google, MicrosoftFoundry
├── Umbraco.AI.Prompt → Prompt.Deploy (depends on Prompt + Deploy)
├── Umbraco.AI.Agent → Agent.UI → Agent.Copilot
│                     → Agent.Deploy (depends on Agent + Deploy)
│                     → Automate (depends on Agent + Automate.Core)
├── Umbraco.AI.Search
└── Umbraco.AI.Deploy
```

### Project Structure Patterns

**Core/Add-on packages** (Umbraco.AI, Agent, Prompt):
```
ProductName/
├── src/
│   ├── ProductName.Core/              # Domain models, services, interfaces
│   ├── ProductName.Web/               # Management API
│   ├── ProductName.Web.StaticAssets/Client/  # TypeScript/Lit frontend
│   ├── ProductName.Persistence/       # EF Core DbContext, repositories
│   ├── ProductName.Persistence.SqlServer/   # SQL Server migrations
│   ├── ProductName.Persistence.Sqlite/      # SQLite migrations
│   ├── ProductName.Startup/           # Umbraco Composer for DI
│   └── ProductName/                   # Meta-package
├── tests/ (Unit, Integration, Common)
├── ProductName.slnx
└── CLAUDE.md
```

**Search package** (Umbraco.AI.Search) uses `Db`, `Db.SqlServer`, `Db.Sqlite` instead of `Persistence.*` — these are `IAIVectorStore` implementations, not domain entity persistence.

**Provider packages**: Single `src/ProviderName/` project + `tests/ProviderName.Tests.Unit/`.

### Core Concepts

- **Providers** - AI service plugins (OpenAI, Anthropic, etc.)
- **Connections** - API keys and provider settings
- **Profiles** - Connection + model settings for use cases
- **Capabilities** - Chat, Embedding, etc.

Built on Microsoft.Extensions.AI (M.E.AI), "thin wrapper" philosophy.

## Development Environment

### Setup

```bash
/repo-setup  # First-time: git hooks, demo site, dependencies, build
```

### Demo Site

**Location:** `demo/Umbraco.AI.DemoSite/` | **Credentials:** admin@example.com / password1234

```bash
/demo-site-management start|stop|open|generate-client|status
/demo-site-automation login|navigate-to-connections|create-connection [provider]
```

- Uses `DemoSite-Claude` profile with dynamic ports (avoids worktree conflicts)
- HTTP over named pipes: `umbraco.demosite.{branch-or-worktree}`
- Site address: query `/site-address` via named pipe to get HTTPS address

### Package Testing Site

Test deployed packages from different feeds (vs demo site which uses project references):

```bash
.\scripts\install-package-test-site.ps1 -Feed nightly|prereleases|release [-SiteName "Name"]
./scripts/install-package-test-site.sh --feed=release --name="Name"  # Linux/Mac
```

Feeds: `nightly` (MyGet nightly), `prereleases` (MyGet pre-release), `release` (NuGet.org). Sites created in `demo/{SiteName}`.

## Project Management (Azure DevOps)

| Purpose | Project | Notes |
|---------|---------|-------|
| Backlog & Work Items | D-Team Tracker | AI Team backlog, tag: `Umbraco AI` |
| CI/CD Pipelines | Umbraco AI | |

**IMPORTANT**: Always scope work item searches to AI Team backlog using `wit_list_backlog_work_items`. The D-Team Tracker project is shared across multiple product teams -- unfiltered searches return cross-product results.

## Build Commands

### .NET

```bash
dotnet build Umbraco.AI.local.slnx          # All products + demo
dotnet build <Product>/<Product>.slnx        # Individual product
dotnet test <Product>/<Product>.slnx         # Run tests
```

### Frontend (npm workspaces)

```bash
npm install                  # All workspace dependencies
npm run build                # All frontends (sequential: core -> prompt -> agent -> agent-ui -> copilot)
npm run watch                # All frontends in parallel
npm run generate-client      # OpenAPI clients (requires running demo site)
npm run build:<target>       # Targets: core, prompt, agent, agent-ui, copilot
npm run watch:<target>       # Same targets as build
```

Add-on packages use `workspace:*` to reference local core during dev; replaced with actual version on `npm pack`.

## Target Framework & Stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umbraco/Umbraco.AI](https://github.com/umbraco/Umbraco.AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
