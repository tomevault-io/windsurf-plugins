---
trigger: always_on
description: This file provides guidance to Coding Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Coding Agents when working with code in this repository.

## Repository Overview

AI Coding Factory is a comprehensive system that automates the enterprise .NET application development lifecycle using specialized OpenCode agents. It implements a multi-agent orchestration model where each agent handles specific development stages (Ideation → Prototype → PoC → Pilot → Product).

## Architecture

This is an **OpenCode-based multi-agent system**, not a traditional code repository. The core architecture consists of:

- **5 Specialized Agents** (`.opencode/agent/`) - Stage-specific AI assistants for development lifecycle
- **11+ Reusable Skills** (`.opencode/skill/`) - Composable workflows for common .NET patterns
- **3 Custom Plugins** (`.opencode/plugin/`) - TypeScript-based integrations (SonarQube, OWASP, CI/CD)
- **Agent Templates** (`.opencode/templates/`) - Agile/Scrum artifacts and role playbooks
- **Project Templates** (`templates/`) - Clean Architecture and microservice boilerplates

### Multi-Agent Development Flow

```
Ideation → Prototype → PoC → Pilot → Product
(reqs)     (MVP)       (secure) (tests) (monitor)
```

Each agent has specific responsibilities, temperature settings (0.1-0.7), and tool permissions configured in `.opencode/opencode.json`.

### Agent Configuration System

Agents are configured with:
- **Mode**: primary (user-facing) or subagent (helper)
- **Model**: Local inference model (GLM-4.7 via vLLM/LM Studio)
- **Temperature**: Creativity level (ideation=0.7, production agents=0.1-0.2)
- **Tools**: Permissions for write, edit, bash operations
- **Skills**: Access to net-* skills for .NET patterns

### Skill System

Skills are markdown files (SKILL.md) with frontmatter metadata. They provide reusable workflows:
- `net-web-api` - ASP.NET Core scaffolding
- `net-domain-model` - DDD patterns
- `net-repository-pattern` - Data access
- `net-jwt-auth` - Authentication
- `net-testing` - Test framework setup
- `net-docker` - Containerization
- `net-github-actions` - CI/CD pipelines
- `net-agile` - Enterprise agile practices
- `net-scrum` - Scrum ceremonies and artifacts
- `net-observability`, `net-kubernetes`, `net-cqrs` - Advanced patterns

Skills are auto-discovered and loaded by agents on-demand.

## Common Commands

### Development Workflow
```bash
# Initial setup (only run once)
./scripts/setup.sh

# Validate project structure
./scripts/validate-project.sh

# Create new project from template
cd projects
mkdir <project-name>
cd <project-name>
opencode
/agent ideation
```

### .NET Development (within generated projects)
```bash
# Restore dependencies
dotnet restore

# Build solution
dotnet build

# Run tests
dotnet test

# Run specific test project
dotnet test tests/<ProjectName>.UnitTests

# Run API project
cd src/<ProjectName>.Api
dotnet run

# Create migration (EF Core)
cd src/<ProjectName>.Infrastructure
dotnet ef migrations add <MigrationName> --startup-project ../ProjectName.Api

# Update database
dotnet ef database update --startup-project ../ProjectName.Api
```

### Docker Operations
```bash
# Build and run with Docker Compose
docker-compose up --build

# Run in detached mode
docker-compose up -d

# Stop containers
docker-compose down

# View logs
docker-compose logs -f api
```

### Agent Commands (within OpenCode)
```bash
# Switch agents
/agent ideation    # Requirements and architecture
/agent prototype   # Rapid MVP development
/agent poc         # Production considerations
/agent pilot       # Production readiness
/agent product     # Maintenance and scaling

# Invoke subagents
@net-security-auditor   # Security vulnerability scan
@net-code-reviewer      # Code quality review
@net-test-generator     # Generate test cases
```

## Key Architecture Patterns

### Clean Architecture Layers
Generated .NET projects follow strict dependency rules:

1. **Domain Layer** (no dependencies)
   - Entities: Business objects with identity
   - Value Objects: Immutable objects compared by value
   - Aggregates: Consistency boundaries with aggregate roots
   - Domain Events: Capture domain changes
   - Repository Interfaces: Data access contracts

2. **Application Layer** (depends on Domain)
   - CQRS: Commands and Queries via MediatR
   - DTOs: API data transfer objects
   - Validators: FluentValidation for input validation
   - Behaviours: Cross-cutting concerns (logging, validation, transactions)

3. **Infrastructure Layer** (depends on Domain + Application)
   - Repository Pattern: IRepository<T> implementations
   - Unit of Work: Transaction management across repositories
   - DbContext: Entity Framework Core configuration
   - External Services: Third-party integrations

4. **API Layer** (depends on Application + Infrastructure)
   - Controllers: RESTful endpoints
   - Middleware: Exception handling, correlation IDs
   - Filters: Authentication, validation
   - Extensions: Dependency injection setup

**Critical**: Infrastructure and API depend on abstractions in Domain/Application. Domain has zero external dependencies.

### Agent-Skill Interaction Pattern

When an agent needs to implement a pattern:
1. Agent identifies required skill (e.g., "net-jwt-auth")
2. OpenCode loads skill's SKILL.md instructions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitkox/ai-coding-factory](https://github.com/mitkox/ai-coding-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
