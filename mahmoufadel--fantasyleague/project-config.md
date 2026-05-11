---
trigger: always_on
description: **This project operates with auto-approval enabled.**
---

# FantasyPremierLeague - Claude Instructions

## Permissions

**This project operates with auto-approval enabled.**

Claude Code has permission to:
- ✅ Read any file in the project
- ✅ Write and edit files
- ✅ Create new files and directories
- ✅ Run dotnet CLI commands (build, test, run)
- ✅ Run npm commands for the frontend
- ✅ Execute bash/shell commands within the project directory
- ✅ Start background processes (API, tests)

**Do not ask for permission again** for routine development tasks in this project.

## Command Execution Preferences

**Proceed with reasonable defaults without asking:**
- When running commands that require timeouts, use appropriate values (30-120s)
- When a command fails, retry once with fixes or try alternative approaches
- When multiple options exist, choose the most common/sensible one
- Don't ask "Should I...?" - just do it and report the result

**Only ask the user when:**
- The operation is destructive (delete, drop, force push)
- Multiple significantly different approaches exist with trade-offs
- The request is genuinely ambiguous
- External credentials or configuration is needed

---

## Project Overview

This is a full-stack **Fantasy Premier League** application built with **.NET 9**, **Entity Framework Core**, and **React 18**. It follows **Domain-Driven Design (DDD)** with Clean Architecture principles.

### Technology Stack
- **Backend**: .NET 9, ASP.NET Core Web API, EF Core (In-Memory Database)
- **Frontend**: React 18, Axios, React Router
- **Testing**: xUnit, Moq, AutoFixture, FluentAssertions
- **API Documentation**: Swagger/OpenAPI

---

## Project Structure

```
FantasyPremierLeague/
├── src/
│   ├── Domain/                 # Core business logic (no external dependencies)
│   │   ├── Common/             # Base classes (Entity, IDomainEvent)
│   │   └── Entities/           # Domain entities (Player, Team, GameWeek, MatchResult)
│   │
│   ├── Application/            # Application services & use cases
│   │   ├── DTOs/               # Data Transfer Objects
│   │   ├── Interfaces/         # Repository & service interfaces
│   │   └── Services/           # Application services
│   │
│   ├── Infrastructure/         # Data access & external services
│   │   ├── Persistence/        # EF Core DbContext, seed data
│   │   └── Repositories/       # Repository implementations
│   │
│   ├── WebApi/                 # API controllers & React frontend
│   │   ├── Controllers/        # API controllers (thin, delegate to services)
│   │   ├── ClientApp/          # React application
│   │   └── Program.cs          # Application entry point
│   │
│   ├── McpServer/              # MCP server library (models & tools)
│   ├── McpServer.Api/          # MCP server HTTP API
│   └── McpServer.Host/         # MCP server host process
│
├── tests/
│   ├── UnitTests/              # Unit tests for domain entities and application services
│   │   └── Domain/Entities/    # Domain entity tests (namespace: UnitTests.Domain.Entities)
│   └── IntegrationTests/       # End-to-end tests using real PostgreSQL via Testcontainers
│       └── Fixtures/           # WebApplicationFactory and container setup
│
└── FantasyPremierLeague.sln    # Solution file
```

---

## Architecture Principles

### Dependency Direction
```
WebApi → Application → Domain
              ↑
        Infrastructure
```

### Layer Responsibilities

| Layer | Responsibility |
|-------|----------------|
| **Domain** | Entities, value objects, aggregates, domain events, business rules |
| **Application** | CQRS (Commands, Queries, Handlers), DTOs, repository interfaces |
| **Infrastructure** | Persistence (EF Core), repository implementations, external services |
| **WebApi** | Controllers, request/response contracts, React SPA |

### Key Rules
- **No business logic in controllers** - Controllers must be thin
- **No persistence logic in domain** - Domain is pure C#, no framework dependencies
- **No framework dependencies in domain layer** - Protect domain from infrastructure concerns

---

## Domain Model

### Core Entities

#### Player (Aggregate Root)
- Properties: `Name`, `Position` (GK, DEF, MID, FWD), `Club`, `Price`, `Points`, `GoalsScored`, `Assists`, `CleanSheets`
- Business rules: Price validation (> 0), stats accumulation
- Points calculation: Goals (×5) + Assists (×3) + CleanSheets (×4)

#### Team (Aggregate Root)
- Properties: `Name`, `Manager`, `Budget` (£100m default), `Points`, `Players` collection
- Business rules: Max 15 players, budget constraints
- Contains `TeamPlayer` value objects (junction entity)

#### GameWeek
- Properties: `WeekNumber`, `StartDate`, `EndDate`, `Status` (Upcoming, Active, Completed)
- Business rules: Activation/completion logic

---

## API Endpoints

### Players
- `GET /api/players` - Get all players
- `GET /api/players/{id}` - Get player by ID
- `GET /api/players/position/{position}` - Get players by position
- `POST /api/players` - Create new player
- `PUT /api/players/{id}/stats` - Update player stats
- `DELETE /api/players/{id}` - Delete player

### Teams
- `GET /api/teams` - Get all teams
- `GET /api/teams/{id}` - Get team by ID
- `POST /api/teams` - Create new team
- `POST /api/teams/add-player` - Add player to team

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mahmoufadel/FantasyLeague](https://github.com/mahmoufadel/FantasyLeague) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
