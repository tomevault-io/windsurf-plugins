---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Pre-Commit Rule

**Before every commit**, run `make lint` and `make typecheck` and fix any errors. Do not commit code that fails linting or type checking.

## Project Overview

Lintel is an open-source AI collaboration infrastructure platform. It orchestrates multi-agent workflows triggered from Slack threads, using event sourcing and CQRS patterns. Built with Python 3.12+, FastAPI, LangGraph, and PostgreSQL.

## Workspace Structure

This is a **uv workspace monorepo** with 45+ packages under `packages/`:

**Core packages:**

| Package | Name | Dependencies | Description |
|---------|------|-------------|-------------|
| `packages/contracts/` | `lintel-contracts` | (none) | Pure domain contracts: types, commands, events, Protocol interfaces |
| `packages/agents/` | `lintel-agents` | contracts | AI agent runtime (roles: planner, coder, reviewer, pm, designer, summarizer) |
| `packages/workflows/` | `lintel-workflows` | contracts, agents | LangGraph workflow orchestration and graph nodes |
| `packages/app/` | `lintel` | all below | Thin composition root: lifespan wiring, middleware, router mounting |

**Reusable library packages (each independently installable):**

| Package | Name | Dependencies | Description |
|---------|------|-------------|-------------|
| `packages/event-store/` | `lintel-event-store` | contracts | Append-only event persistence (Postgres + in-memory) |
| `packages/event-bus/` | `lintel-event-bus` | contracts | In-memory pub/sub event bus |
| `packages/projections/` | `lintel-projections` | contracts, event-bus | Read-model projection engine and concrete projections |
| `packages/persistence/` | `lintel-persistence` | contracts | Generic CRUD/dict stores, vault |
| `packages/sandbox/` | `lintel-sandbox` | contracts | Docker-based code execution sandboxes |
| `packages/pii/` | `lintel-pii` | contracts | PII detection/anonymization (presidio) |
| `packages/observability/` | `lintel-observability` | contracts | OpenTelemetry tracing/metrics/logging |
| `packages/models/` | `lintel-models` | contracts | LLM provider routing (litellm) |
| `packages/slack/` | `lintel-slack` | contracts | Slack channel adapter |
| `packages/repos/` | `lintel-repos` | contracts | GitHub repository provider |
| `packages/coordination/` | `lintel-coordination` | (asyncpg only) | Database advisory locks |
| `packages/infrastructure/` | `lintel-infrastructure` | (residual) | MCP tool client only — will be dissolved |

**API route packages (extracted from app — each independently installable):**

| Package | Name | Namespace | Description |
|---------|------|-----------|-------------|
| `packages/api-support/` | `lintel-api-support` | `lintel.api_support` | `StoreProvider`, `dispatch_event`, `EntityStore`/`DictStore` protocols |
| `packages/users/` | `lintel-users` | `lintel.users` | User CRUD routes + in-memory store |
| `packages/teams/` | `lintel-teams` | `lintel.teams` | Team CRUD routes + in-memory store |
| `packages/policies-api/` | `lintel-policies-api` | `lintel.policies_api` | Policy CRUD routes + in-memory store |
| `packages/notifications-api/` | `lintel-notifications-api` | `lintel.notifications_api` | Notification rule CRUD routes + in-memory store |
| `packages/environments-api/` | `lintel-environments-api` | `lintel.environments_api` | Environment CRUD routes + in-memory store |
| `packages/variables-api/` | `lintel-variables-api` | `lintel.variables_api` | Variable CRUD routes + in-memory store |
| `packages/credentials-api/` | `lintel-credentials-api` | `lintel.credentials_api` | Credential CRUD routes + in-memory store |
| `packages/audit-api/` | `lintel-audit-api` | `lintel.audit_api` | Audit entry CRUD routes + in-memory store |
| `packages/approval-requests-api/` | `lintel-approval-requests-api` | `lintel.approval_requests_api` | Approval request CRUD routes + in-memory store |
| `packages/boards/` | `lintel-boards` | `lintel.boards` | Board + tag CRUD routes + in-memory stores |
| `packages/triggers-api/` | `lintel-triggers-api` | `lintel.triggers_api` | Trigger CRUD routes + in-memory store |
| `packages/artifacts-api/` | `lintel-artifacts-api` | `lintel.artifacts_api` | Code artifact + test result routes + in-memory stores |
| `packages/projects-api/` | `lintel-projects-api` | `lintel.projects_api` | Project CRUD routes + in-memory store |
| `packages/work-items-api/` | `lintel-work-items-api` | `lintel.work_items_api` | Work item CRUD routes + in-memory store |
| `packages/skills-api/` | `lintel-skills-api` | `lintel.skills_api` | Skill CRUD routes + in-memory store |
| `packages/agent-definitions-api/` | `lintel-agent-definitions-api` | `lintel.agent_definitions_api` | Agent definition CRUD routes + in-memory store |
| `packages/mcp-servers-api/` | `lintel-mcp-servers-api` | `lintel.mcp_servers_api` | MCP server CRUD routes + in-memory store |
| `packages/models-api/` | `lintel-models-api` | `lintel.models_api` | Model CRUD routes + in-memory store |
| `packages/ai-providers-api/` | `lintel-ai-providers-api` | `lintel.ai_providers_api` | AI provider CRUD routes + in-memory store |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bamdadd/lintel](https://github.com/bamdadd/lintel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
