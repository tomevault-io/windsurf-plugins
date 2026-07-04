---
trigger: always_on
description: <!-- SPDX-FileCopyrightText: 2026 Aryan Iyappan <aryaniyappan2006@gmail.com> -->
---

<!-- SPDX-FileCopyrightText: 2026 Aryan Iyappan <aryaniyappan2006@gmail.com> -->
<!-- SPDX-FileCopyrightText: 2026 Hari Srinivasan <harisrini21@gmail.com> -->
<!-- SPDX-FileCopyrightText: 2026 Lokesh Selvam <lokeshselvam7025@gmail.com> -->
<!-- SPDX-License-Identifier: AGPL-3.0-only -->

# AGENTS.md

Internal context for contributors and AI coding agents. Use `README.md` for the public API reference, `SETUP.md` for environment setup, and `docs/adding-a-harness.md` for harness integration.

## What Observal is

Observal is an agent-centric registry and observability platform for AI coding agents. Users interact with it three ways:

1. **CLI** (`observal`): pull agents, sca harnesses, submit components, manage the server
2. **Web UI** (`web/`): browse the registry, view traces, manage users, admin dashboard
3. **Observal skill** (bundled, auto-installed on login): lets the LLM inside any harness drive Observal commands directly (e.g. "create an agent that uses the github MCP")

Agents are the primary entity. Each agent bundles 5 component types: MCP servers, skills, hooks, prompts, and sandboxes. When a user runs `observal pull <agent>`, the platform resolves all components and writes harness-specific config files.

## harness support tiers

**First-class** (full session parsing, hooks, scanning, config gen, tested e2e):
- Claude Code
- Kiro
- Cursor
- Pi

**Functional** (config gen and scanning work, but no session parser or hook spec):
- Codex CLI, Copilot, Copilot CLI, OpenCode

See `docs/adding-a-harness.md` for the complete guide to adding or promoting a harness.

## Architecture at a glance

```
observal_cli/          Python CLI (Typer)
  harness/             CLI-side harness adapters (protocol.py, base.py, 9 adapters)
  harness_specs/           Hook specs (claude_code, kiro, pi only)
  skills/              Bundled skills installed on login (observal, observal-admin, etc.)

observal-server/       FastAPI server
  api/routes/          REST endpoints (agent/, admin/ are sub-packages)
  api/middleware/      Audit, request-id, content-type
  models/              SQLAlchemy models (PostgreSQL)
  schemas/             Pydantic request/response schemas
  services/            Business logic
    clickhouse/        ClickHouse subpackage (client, schema, insert, query)
    harness/           Server-side harness adapters (config generation)
    session_parsers/   Per-harness JSONL parsers (claude_code, kiro, cursor, pi)
    audit/             Compliance audit system (loguru-based)
    config/            Config generation helpers (mcp_builder, skill_builder)
    insights/          Insight engine (report generation, facets, sections, HTML export)
    shared/            Cross-service utilities
  jobs/                Background job definitions (catalog, maintenance)

ee/                    Enterprise (source-available, separate license)
  license.py           JWT license validation
  observal_server/     EE routes + services (audit, SAML, SCIM, exec dashboard)

web/                   Next.js 16 / React 19 frontend
packages/pi-extension/ Pi telemetry extension (npm: observal-pi)
docker/                Docker Compose stack (10 services)
tests/                 pytest (123 files)
tests/e2e/             Playwright (19 specs)
```

## How the modularisation works

The codebase follows a strict adapter pattern for harness-specific logic. This is the most important architectural decision:

**One adapter per harness, on both sides.** CLI adapters handle scanning and hook detection (`observal_cli/harness/<name>.py`). Server adapters handle config file generation (`observal-server/services/harness/<name>.py`). The harness registry (`harness_registry.py`, mirrored on both sides) defines paths, keys, features, and event maps.

**No if/elif chains for harness logic.** If you need harness-specific behavior, it goes in the adapter. The orchestrators (`cmd_scan.py`, `agent_builder.py`, `cmd_doctor.py`) call adapters via the registry, never with conditionals.

**Feature-flag gating.** Each adapter method maps to a feature (`hooks`, `mcp_servers`, `skills`). The `BaseAdapter` raises `NotSupportedError` if the harness's registry entry lacks the required feature. This means stubs are safe: they exist but can't be called for unsupported operations.

**Session parsers are separate from adapters.** They live in `services/session_parsers/` (server-side) and handle converting raw JSONL into normalized trace events. Only first-class harnesses have parsers.

### What "first-class" means concretely

A first-class harness has all of:
- A hook spec in `harness_specs/` (defines what `doctor patch --hook` installs)
- A session parser in `services/session_parsers/` (enables `observal reconcile`)
- Full scanning implementation in its CLI adapter (discovers MCPs, skills, hooks, agents)
- E2E test coverage in `tests/e2e/`

A stub harness has:
- A registry entry with correct paths
- A CLI adapter that handles basic MCP scanning
- A server adapter that generates config files
- No hook spec, no session parser, no e2e tests

## Coding patterns we prefer

### Python (server + CLI)

- **Ruff** for lint and format. Line length 120. Pre-commit enforces it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Observal/Observal](https://github.com/Observal/Observal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
