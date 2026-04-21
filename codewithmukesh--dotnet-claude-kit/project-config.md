---
trigger: always_on
description: > This file defines how Claude Code routes queries to specialist agents and how agents coordinate.
---

# Agent Routing & Orchestration

> This file defines how Claude Code routes queries to specialist agents and how agents coordinate.

## Agent Roster

| Agent | File | Primary Domain |
|-------|------|---------------|
| dotnet-architect | `agents/dotnet-architect.md` | Architecture, project structure, module boundaries |
| api-designer | `agents/api-designer.md` | Minimal APIs, OpenAPI, versioning, rate limiting |
| ef-core-specialist | `agents/ef-core-specialist.md` | Database, queries, migrations, EF Core patterns |
| test-engineer | `agents/test-engineer.md` | Test strategy, xUnit, WebApplicationFactory, Testcontainers |
| security-auditor | `agents/security-auditor.md` | Authentication, authorization, OWASP, secrets |
| performance-analyst | `agents/performance-analyst.md` | Benchmarks, memory, async patterns, caching |
| devops-engineer | `agents/devops-engineer.md` | Docker, CI/CD, Aspire, deployment |
| code-reviewer | `agents/code-reviewer.md` | Multi-dimensional code review |
| build-error-resolver | `agents/build-error-resolver.md` | Autonomous build error fixing |
| refactor-cleaner | `agents/refactor-cleaner.md` | Systematic dead code removal and cleanup |

## Routing Table

Match user intent to agent. When multiple agents could handle a query, the first match wins.

| User Intent Pattern | Primary Agent | Support Agent |
|---|---|---|
| "set up project", "folder structure", "architecture" | dotnet-architect | — |
| "add module", "split into modules", "bounded context" | dotnet-architect | — |
| "create endpoint", "API route", "OpenAPI", "swagger" | api-designer | — |
| "versioning", "rate limiting", "CORS" | api-designer | — |
| "database", "migration", "query", "DbContext", "EF" | ef-core-specialist | — |
| "write tests", "test strategy", "coverage" | test-engineer | — |
| "WebApplicationFactory", "Testcontainers", "xUnit" | test-engineer | — |
| "security", "authentication", "JWT", "OIDC", "authorize" | security-auditor | — |
| "performance", "benchmark", "memory", "profiling" | performance-analyst | — |
| "caching", "HybridCache", "output cache" | performance-analyst | — |
| "Docker", "container", "CI/CD", "pipeline", "deploy" | devops-engineer | — |
| "Aspire", "orchestration", "service discovery" | devops-engineer | — |
| "review this code", "PR review", "code quality" | code-reviewer | — |
| "choose architecture", "which architecture", "architecture decision" | dotnet-architect | — |
| "scaffold feature", "create feature", "add endpoint", "generate feature" | dotnet-architect | api-designer, ef-core-specialist |
| "init project", "setup project", "new project", "generate CLAUDE.md" | dotnet-architect | — |
| "health check", "analyze project", "project report" | code-reviewer | dotnet-architect |
| "review PR", "review changes", "code review", "PR review" | code-reviewer | — |
| "add migration", "ef migration", "update packages", "upgrade nuget" | ef-core-specialist | — |
| "conventions", "coding style", "detect patterns", "code consistency" | code-reviewer | — |
| "add feature" (architecture-appropriate) | dotnet-architect | api-designer, ef-core-specialist |
| "refactor" | code-reviewer | dotnet-architect |
| "build errors", "fix build", "won't compile" | build-error-resolver | — |
| "clean up", "dead code", "unused code", "de-sloppify" | refactor-cleaner | — |

## Skill Loading Order

Agents load skills in dependency order. Core skills load first.

### Default Load Order (All Agents)
1. `modern-csharp` — Always loaded, baseline C# knowledge
2. Agent-specific skills (see agent files)

### Per-Agent Skill Maps

| Agent | Skills |
|-------|--------|
| dotnet-architect | modern-csharp, architecture-advisor, project-structure, scaffolding, project-setup + conditional: vertical-slice, clean-architecture, ddd |
| api-designer | modern-csharp, minimal-api, api-versioning, authentication, error-handling |
| ef-core-specialist | modern-csharp, ef-core, configuration, migration-workflow |
| test-engineer | modern-csharp, testing |
| security-auditor | modern-csharp, authentication, configuration |
| performance-analyst | modern-csharp, caching |
| devops-engineer | modern-csharp, docker, ci-cd, aspire |
| code-reviewer | modern-csharp, code-review-workflow, convention-learner + contextual (loads relevant skills incl. clean-architecture, ddd based on files under review) |
| build-error-resolver | modern-csharp, autonomous-loops + contextual: ef-core, dependency-injection |
| refactor-cleaner | modern-csharp, de-sloppify + contextual: testing, ef-core |

## MCP Tool Preferences

Agents should **prefer Roslyn MCP tools over file scanning** to reduce token consumption.

| Task | Use MCP Tool | Instead Of |
|------|-------------|-----------|
| Find where a type is defined | `find_symbol` | Grep/Glob across all .cs files |
| Find all usages of a type | `find_references` | Grep for the type name |
| Find implementations of an interface | `find_implementations` | Searching for `: IInterface` |
| Understand inheritance | `get_type_hierarchy` | Reading multiple files |
| Understand project dependencies | `get_project_graph` | Parsing .csproj files manually |
| Review a type's API surface | `get_public_api` | Reading the full source file |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codewithmukesh/dotnet-claude-kit](https://github.com/codewithmukesh/dotnet-claude-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
