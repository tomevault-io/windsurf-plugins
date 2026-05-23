---
trigger: always_on
description: This file provides AI assistant guidance for working with the ToolHive Registry API Server codebase.
---

# CLAUDE.md

This file provides AI assistant guidance for working with the ToolHive Registry API Server codebase.

## Design Rules

Scoped design rules live in [`.claude/rules/`](.claude/rules/). Each file has a `paths:` frontmatter glob and is auto-loaded when Claude Code is working on files that match. They cover layering, the data model, auth/claims, the API surface (admin vs upstream spec), the sync pipeline, and secrets handling. Read the rule file that matches the area you're changing, and cite it by section when reviewing a PR that conflicts with one (e.g. "this violates `auth.md` §3 — uniform claim matching").

## Available Subagents

Registry Server uses specialized AI subagents for different aspects of development. These agents are configured in `.claude/agents/` and MUST be invoked when you need to perform tasks that come under their expertise:

### Core Development Agents

- **golang-code-writer**: Expert Go developer for writing clean, idiomatic Go code. Use when creating new functions, structs, interfaces, or complete packages.

- **unit-test-writer**: Specialized in writing comprehensive unit tests for Go code. Use when you need thorough test coverage for functions, methods, or components.

- **code-reviewer**: Reviews code for Registry Server best practices, security patterns, Go conventions, and architectural consistency. Use after significant code changes. Should cross-check changes against the matching rule file under `.claude/rules/`.

- **tech-lead-orchestrator**: Provides architectural oversight, task delegation, and technical leadership for code development projects. Use for complex features or architectural decisions.

### Support Agents

- **security-advisor**: Provides security guidance for coding tasks, including code reviews, architecture decisions, and secure implementation patterns.

### When to Use Subagents

Invoke specialized agents when:
- You need expertise in their specific domain (e.g., writing code, reviewing code)
- Writing new code (use golang-code-writer)
- Creating tests (use unit-test-writer)
- Orchestrating the different tasks to other subagents that are required for completing work asked by the user (use tech-lead-orchestrator)
- Reviewing code that is written (use code-reviewer)

The agents work together - for example, tech-lead-orchestrator might delegate to golang-code-writer for implementation and then to code-reviewer for validation, then finally to security-advisor for a security check of any implementation details.

## Project Overview

See [README.md](README.md) for complete project documentation including features, API endpoints, configuration, and deployment.

**Quick Summary**: A standards-compliant MCP Registry API server that provides REST endpoints for discovering MCP servers and skills. Ingests data from Git, API, file, and Kubernetes sources into a shared entry pool, and exposes consumer views through claim-scoped registries.

## Essential Build Commands

See [README.md](README.md#development) for complete build and development documentation.

**Most commonly used:**
- `task build` - Build the binary
- `task lint-fix` - **Preferred** linting (auto-fixes issues)
- `task test` - Run tests
- `task gen` - Generate mocks (run before testing)
- `task all` - Lint, test, and build

## Code Architecture

High-level shape (read `.claude/rules/layering.md` for the rules that govern layer boundaries):

```
cmd/thv-registry-api/      # Entrypoint only: main.go, app wiring, generated OpenAPI docs
internal/
  api/                     # HTTP layer — Chi routers and handlers
    v1/                      # Admin API (/v1/sources, /v1/registries, /v1/entries, /v1/me)
    registry/v01/            # Upstream MCP Registry v0.1 consumer API (/registry/{name}/v0.1/...)
    x/skills/                # Consumer extension: /x/dev.toolhive/skills
    common/                  # Shared request/response helpers
  app/                     # Application assembly and lifecycle (builder + functional options)
    storage/                 # storage.Factory — DI seam for all DB-backed components
  service/                 # Business logic — RegistryService interface, claim checks, dedup, cursors
    db/                      # Postgres-backed service implementation and claims filter
  sources/                 # Ingestion handlers — RegistryHandler, RegistryHandlerFactory (git, api, file)
  sync/                    # Sync orchestration — Manager, Coordinator, writer (serializable txn)
  kubernetes/              # K8s controllers — MCPServer / VirtualMCPServer / MCPRemoteProxy CRDs
  auth/                    # Authentication (JWT middleware, validators, OAuth config)
  authz/                   # Authorization integration tests (role + claim enforcement)
  audit/                   # Audit logging middleware (wraps all /v1 handlers)
  config/                  # YAML config loader and validation (viper)
  db/                      # Low-level DB plumbing, migrations runner, sqlc-generated code
  filtering/               # Name/tag filter implementation (used during ingestion)
  telemetry/               # OpenTelemetry setup (metrics + tracing)
database/migrations/       # SQL migrations (numbered, up/down)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stacklok/toolhive-registry-server](https://github.com/stacklok/toolhive-registry-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
