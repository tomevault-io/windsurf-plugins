---
trigger: always_on
description: Rules for coordinating multiple agents running simultaneously in Cursor 2.0
---


# 🤖 Multi-Agent Coordination (Cursor 2.0)

## Overview

Cursor 2.0 supports **up to 8 agents running concurrently**. Each agent operates in its own isolated copy of the codebase using git worktrees or remote machines to prevent file conflicts.

## Multi-Agent Access

To enable multiple agents:
1. **Open Cursor 2.0** (requires latest version)
2. **Multi-Agent Sidebar**: New sidebar interface for managing agents and plans
3. **Activate Agents**: Use the agent management UI to spawn multiple agents
4. **Assign Tasks**: Each agent can work on different parts of the codebase simultaneously

## Agent Role Scoping

Define agent responsibilities using path patterns to prevent conflicts and ensure efficient parallel execution:

### 🎯 Service-Specific Agents

```yaml
Frontend_Agent:
  scope: 
    - path: "v2/frontend/**"
    - path: "v2/shared/ui/**"
  responsibilities:
    - UI/UX improvements
    - Frontend testing with Browser Agent
    - React component development
    - State management updates
  tools_priority:
    - Browser Agent (E2E testing)
    - mcp_filesystem_* (config files)
    - mcp_postgres_* (read-only data validation)

Backend_Service_Agent:
  scope:
    - path: "v2/*_service/**"
    - exclude: "v2/frontend/**"
    - exclude: "v2/db_service/migrations/**"
  responsibilities:
    - Service implementation
    - API endpoint development
    - Business logic
    - Integration testing
  tools_priority:
    - mcp_loki_* (service logs)
    - mcp_postgres_* (data operations)
    - mcp_repo-indexer_* (code patterns)

Database_Agent:
  scope:
    - path: "v2/db_service/migrations/**"
    - path: "v2/db_service/**"
  responsibilities:
    - Migration creation ONLY in v2/db_service/migrations/
    - Schema changes
    - Database optimization
    - Query performance
  tools_priority:
    - mcp_postgres_* (schema operations)
    - mcp_filesystem_* (migration files)
  critical: "NEVER creates migrations outside v2/db_service/migrations/"

Observability_Agent:
  scope:
    - path: "v2/observability_stack/**"
    - path: "v2/*/grafana/**"
    - path: "v2/*/prometheus/**"
  responsibilities:
    - Grafana dashboards
    - Prometheus metrics
    - Alert rules (remember: down -v before up -d!)
    - Loki queries for RCA
  tools_priority:
    - mcp_loki_* (log investigation)
    - mcp_filesystem_* (config files)
    - mcp_config-indexer_* (alert configurations)

Documentation_Agent:
  scope:
    - path: "v2/docs/**"
    - path: "*.md"
    - exclude: "v2/docs/plans/**" # Plans are temporary
  responsibilities:
    - Documentation updates (UPDATE existing, don't create new)
    - RCA documents (ONE per incident)
    - API documentation
    - Architecture diagrams
  tools_priority:
    - mcp_docs-indexer_* (search existing docs)
    - mcp_filesystem_* (doc files)
  critical: "Minimize .md creation - update existing files"

Testing_Agent:
  scope:
    - path: "**/tests/**"
    - path: "**/test_*.py"
    - path: "**/*.test.ts"
    - path: "**/*.test.tsx"
  responsibilities:
    - TDD test creation
    - Test refactoring
    - Coverage improvements
    - E2E test scenarios
  tools_priority:
    - Cursor Browser Agent (frontend E2E)
    - mcp_terminal_* (pytest execution)
    - mcp_postgres_* (test data validation)

Infrastructure_Agent:
  scope:
    - path: "docker-compose*.yml"
    - path: ".env*"
    - path: "v2/**/Dockerfile*"
    - path: "v2/**/requirements.txt"
  responsibilities:
    - Container orchestration
    - Environment configuration
    - Dependency management
    - Build pipeline
  tools_priority:
    - mcp_config-indexer_* (Docker analysis)
    - mcp_terminal_* (container commands)
    - mcp_filesystem_* (config files)
```

## Coordination Patterns

### ✅ Parallel Safe Operations

These can run simultaneously across agents:

```yaml
Safe_Parallel_Tasks:
  - Different services (trade_engine + signal_engine)
  - Frontend + Backend (different file paths)
  - Documentation + Code (separate directories)
  - Testing + Development (separate branches/worktrees)
  - Multiple test files (no shared state)
```

### ⚠️ Requires Coordination

These operations need explicit coordination:

```yaml
Coordinate_Before_Executing:
  - Database migrations (only Database_Agent)
  - Shared utility changes (coordinate via TODO or GitHub Issue)
  - API contract changes (affects multiple services)
  - Breaking changes to shared libraries
  - Environment variable changes (.env files)
```

### 🔴 Conflict Zones

Prevent multiple agents from touching:

```yaml
Exclusive_Access_Required:
  - v2/db_service/migrations/ (Database_Agent ONLY)
  - docker-compose.v2.yml (Infrastructure_Agent coordination)
  - Shared library files (v2/shared/** - coordinate first)
  - Critical config files (.env, secrets)
```

## Communication Patterns

### Agent-to-Agent Coordination

```yaml
Coordination_Methods:
  1. GitHub_Issues:
     - Create issue for shared work
     - Agents check issues before starting
     - Update issue with progress
  
  2. TODO_Lists:
     - Use todo_write tool for multi-agent tasks
     - Agents can check and update TODO status
     - Clear task ownership in TODO items
  
  3. Path_Exclusion:
     - Use exclude patterns in agent scopes
     - Prevents accidental overlap
  
  4. Branch_Strategy:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
