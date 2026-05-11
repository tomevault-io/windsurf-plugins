---
trigger: always_on
description: This document defines how AI assistants collaborate effectively in your codebase.
---

# AGENTS.md — Multi-Agent Collaboration Guide

This document defines how AI assistants collaborate effectively in your codebase.

## Goals

- Eliminate confusion about paths, environment, and contracts
- Use each agent's strengths where they shine
- Provide repeatable handoffs and validation gates
- Maintain transparent collaboration for users

## Roles & Strengths

### Primary Orchestrator (Claude Code)
- Architecture and requirements analysis
- Endpoint discovery and contract mapping
- Security and privacy review
- Refactor planning and design
- Risk assessment and validation
- Complex reasoning and debugging

### Implementation Specialist (Codex/GPT)
- Focused implementation
- Surgical edits aligned with style
- Test generation
- Lint and type fixes
- Small migrations
- Keeping changes minimal

### Code Generation (Aider)
- Large-scale file modifications
- Pattern-based transformations
- Repository-wide updates
- Boilerplate generation

## Collaboration Visibility

### Announcement Format
When delegating between agents:
```
🤝 Working with [Agent] on: [task description]
   Reason: [why this agent is suited for this]
   Expected outcome: [what will be delivered]
```

### Progress Tracking
During collaboration:
- `🔄 [Agent] implementing...` - Work in progress
- `✅ [Agent] completed: [summary]` - Task done
- `🔍 Reviewing output...` - Validation phase
- `⚠️ Adjusting: [issue]` - Corrections needed

## Handoff Template

Use this template when delegating tasks between agents:

```yaml
task:
  description: "Brief task description"
  agent: "target-agent-name"
  
context:
  environment: "development|staging|production"
  constraints: ["list", "of", "constraints"]
  dependencies: ["required", "services"]
  
files:
  modify: ["path/to/file1.ts", "path/to/file2.py"]
  create: ["path/to/newfile.ts"]
  delete: ["path/to/obsolete.ts"]
  
contracts:
  endpoints:
    - method: "POST"
      path: "/api/resource"
      request: "{ field: type }"
      response: "{ field: type }"
  
plan:
  - step: "First action"
    acceptance: "What success looks like"
  - step: "Second action"
    acceptance: "What success looks like"
    
validation:
  commands:
    - "npm test"
    - "npm run lint"
  expected: "All tests passing, no lint errors"
  
sandbox:
  filesystem: "read-only|workspace-write|full-access"
  network: "restricted|approved|full"
  approval: "always|on-failure|never"
```

## Example Handoff

```
🤝 Working with Codex on: JWT refresh token implementation
   Reason: Standard API endpoint following established patterns
   Expected outcome: Endpoint, service method, and tests

task:
  description: "Add refresh token endpoint"
  agent: "codex"
  
context:
  environment: "development"
  constraints: ["FastAPI patterns", "80% test coverage"]
  
files:
  modify: 
    - "api/routers/auth.py"
    - "core/services/auth_service.py"
  create:
    - "tests/test_auth_refresh.py"
    
contracts:
  endpoints:
    - method: "POST"
      path: "/api/auth/refresh"
      request: "{ refresh_token: string }"
      response: "{ access_token: string, expires_in: number }"
      
plan:
  - step: "Implement service method"
    acceptance: "Validates token, generates new access token"
  - step: "Add router endpoint"
    acceptance: "Follows FastAPI patterns, proper error handling"
  - step: "Write tests"
    acceptance: "Unit and integration tests, 80%+ coverage"
    
validation:
  commands:
    - "pytest tests/test_auth_refresh.py"
    - "npm run lint"
  expected: "All tests pass, no lint errors"

🔄 Codex implementing...
✅ Codex completed: JWT refresh endpoint with tests
🔍 Reviewing Codex output...
```

## Collaboration Workflows

### 1. Sequential Pipeline
Most common for feature implementation:
```
Claude → Codex → Claude → User
Design → Build → Review → Deliver
```

### 2. Parallel Divide & Conquer
For independent components:
```
     Claude → [Business Logic]
    ↙      ↘
User         → Merge → User
    ↘      ↗
     Codex → [UI/Tests/Docs]
```

### 3. Consensus for Risky Changes
For critical modifications:
```
Claude + Codex → Design Options → Select Best → Implement → Review
```

### 4. Debug Team
For complex issues:
```
Claude: Root cause analysis
  ↓
Codex: Implement fix
  ↓
Claude: Validate security
  ↓
Codex: Add regression tests
```

## Path Discipline

### Project Structure
```yaml
# Define your project structure
paths:
  source: "./src"           # Main source code
  tests: "./tests"          # Test files
  docs: "./docs"            # Documentation
  config: "./config"        # Configuration files
  
# Container paths if using Docker
container:
  app: "/app"
  data: "/data"
  logs: "/var/log/app"
```

### Path References in Handoffs
Always use consistent path references:
- Relative from project root: `./src/components/Button.tsx`
- Absolute when needed: `/app/src/components/Button.tsx`
- Include both host and container paths for Docker projects

## Validation Gates

### Common Commands
```bash
# Testing
npm test                    # Run all tests
npm run test:unit          # Unit tests only
npm run test:integration   # Integration tests
npm run test:e2e           # End-to-end tests
npm run coverage           # Test coverage report

# Code Quality
npm run lint               # Lint code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scottweiss/claude-codex-mcp-starter](https://github.com/scottweiss/claude-codex-mcp-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
