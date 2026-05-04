---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Project Overview

**Clinic Management System** - A layered monolith built on ABP Framework using Domain Driven Design. Manages patients, appointments, and doctor schedules.

**Tech Stack**

| Component | Technology |
|-----------|------------|
| Runtime | .NET 10 |
| Framework | ABP Framework 10.0.1 |
| Database | PostgreSQL |
| ORM | Entity Framework Core |
| Auth | OpenIddict (OAuth 2.0) |
| Mapping | Mapperly (NOT AutoMapper) |
| Validation | FluentValidation (NOT data annotations) |
| Testing | xUnit + Shouldly + NSubstitute |

## Git Rules — STRICT MODE

### NEVER
- `push --force`, `reset --hard`, `clean -fd`, or delete branches
- Commit to `main`, `master`, or protected branches
- Rebase shared branches
- Commit secrets, `.env`, or `appsettings.*.json` with sensitive data

### ALWAYS
- Create feature branch: `ai/<description>`
- Check: `git status`, `git diff`, verify branch before commit
- Pull (no rebase) before push
- Push only feature branches

### COMMIT FORMAT
```
<type>: <what>

Why:
* <reason>
```

### STOP IF
- On protected branch
- Git errors/conflicts
- Unclear state

> **Safety > Speed. Agent is not the repo owner.**

## Quick Reference

| Action | Command |
|--------|---------|
| Build | `dotnet build api/ClinicManagementSystem.slnx` |
| Run API | `dotnet run --project api/src/ClinicManagementSystem.HttpApi.Host` |
| Run Migrations | `dotnet run --project api/src/ClinicManagementSystem.DbMigrator` |
| Run Tests | `dotnet test api/` |

For detailed commands and project structure, see **[docs/architecture/README.md](docs/architecture/README.md)**.

## Claude Code Extensions

For choosing between Agents, Skills, Commands, and Hooks, see **[.claude/GUIDELINES.md](.claude/GUIDELINES.md)**.

| Mechanism | Invocation | Best For |
|-----------|------------|----------|
| **Skill** | Automatic | Domain expertise, patterns |
| **Agent** | Delegated | Complex tasks with context isolation |
| **Command** | `/cmd` | Atomic, frequent actions |

**Creating artifacts**: Say "create a skill for..." or "create an agent that..." to auto-trigger the `claude-artifact-creator` skill.

## Quick Skill Reference

**Use directly for single-skill tasks (no file reads needed):**

| Task | Skill |
|------|-------|
| Entity/DTO/AppService | `abp-framework-patterns` |
| DbContext/Migration | `efcore-patterns` |
| Input validation | `fluentvalidation-patterns` |
| Permissions/Auth | `openiddict-authorization` |
| Unit/Integration tests | `xunit-testing-patterns` |
| Query optimization | `linq-optimization-patterns` |
| API design | `api-design-principles` + `technical-design-patterns` |
| Debug/errors | `debugging-patterns` |
| Security audit | `security-patterns` |
| React components | `react-development-patterns` |

**For multi-skill tasks**: Read [SKILL-INDEX.md](.claude/SKILL-INDEX.md) and [CONTEXT-GRAPH.md](.claude/CONTEXT-GRAPH.md).

**For full Knowledge Discovery Protocol**: See [GUIDELINES.md](.claude/GUIDELINES.md#knowledge-architecture).

## Available Agents

Located in `.claude/agents/` organized by role. Full reference: [.claude/AGENT-QUICK-REF.md](.claude/AGENT-QUICK-REF.md)

### Quick Agent Selection

| Task | Agent |
|------|-------|
| Analyze requirements | `business-analyst` |
| Design API/schema | `backend-architect` |
| Implement .NET/ABP | `abp-developer` |
| Implement React | `react-developer` |
| Review backend code | `abp-code-reviewer` |
| Review frontend code | `react-code-reviewer` |
| Security audit | `security-engineer` |
| Write tests | `qa-engineer` |
| Debug errors | `debugger` |
| DB migrations | `database-migrator` |
| CI/CD setup | `devops-engineer` |

### Common Agent Chains

| Workflow | Chain |
|----------|-------|
| Feature (full) | `business-analyst` → `backend-architect` → `abp-developer` → `qa-engineer` → `abp-code-reviewer` |
| Feature (fast) | `backend-architect` → `abp-developer` → `qa-engineer` |
| Full-stack | `backend-architect` → [`abp-developer`, `react-developer`] → `qa-engineer` → [`abp-code-reviewer`, `react-code-reviewer`] |
| Bug fix | `debugger` → `abp-developer` → `qa-engineer` |

**Usage**: `Use the abp-developer agent to implement the Patient service`

## Available Skills

Located in `.claude/skills/` organized by topic:

| Category | Count | Key Skills |
|----------|-------|------------|
| **Backend** | 12 | abp-framework-patterns, abp-entity-patterns, abp-service-patterns, abp-infrastructure-patterns, efcore-patterns, fluentvalidation-patterns |
| **Microservices** | 3 | distributed-events-advanced, grpc-integration-patterns, bulk-operations-patterns |
| **API Design** | 2 | api-response-patterns, api-design-principles |
| **Requirements** | 5 | requirements-engineering, domain-modeling, technical-design-patterns |
| **Testing** | 3 | xunit-testing-patterns, e2e-testing-patterns, javascript-testing-patterns |
| **Security** | 1 | security-patterns |
| **Frontend** | 3 | react-development-patterns, typescript-advanced-types, modern-javascript-patterns |
| **DevOps** | 2 | docker-dotnet-containerize, git-advanced-workflows |
| **Meta** | 4 | claude-artifact-creator, feature-development-workflow, knowledge-discovery, markdown-optimization |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thapaliyabikendra/ai-artifacts](https://github.com/thapaliyabikendra/ai-artifacts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
