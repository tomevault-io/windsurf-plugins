---
trigger: always_on
description: HyperDX is an observability platform that helps engineers search, visualize, and
---

# HyperDX Development Guide

## What is HyperDX?

HyperDX is an observability platform that helps engineers search, visualize, and
monitor logs, metrics, traces, and session replays. It's built on ClickHouse for
blazing-fast queries and supports OpenTelemetry natively.

**Core value**: Unified observability with ClickHouse performance,
schema-agnostic design, and correlation across all telemetry types in one place.

## Architecture (WHAT)

This is a **monorepo** with six packages:

- `packages/app` - Next.js frontend (TypeScript, Mantine UI, TanStack Query)
- `packages/api` - Express backend (Node.js 22+, MongoDB for metadata,
  ClickHouse for telemetry). Also hosts the **MCP server**, **External API v2**,
  and **OpAMP server** as sub-applications.
- `packages/common-utils` - Shared TypeScript utilities for query parsing and
  validation
- `packages/cli` - Terminal CLI and interactive TUI (`hdx`) for searching,
  tailing, and inspecting logs and traces (Ink/React). Has its own
  [`AGENTS.md`](packages/cli/AGENTS.md) with detailed architecture and
  keybindings.
- `packages/otel-collector` - Custom-built OpenTelemetry Collector (Go, OCB).
  See its [`README.md`](packages/otel-collector/README.md) for architecture,
  included components, and upgrade procedures.
- `packages/hdx-eval` - AI eval framework for benchmarking MCP servers against
  observability scenarios. Generates deterministic synthetic telemetry, spawns
  agents, and grades with programmatic checks + LLM-as-judge. See its
  [`README.md`](packages/hdx-eval/README.md) for setup and usage, and
  [`agent_docs/evals.md`](agent_docs/evals.md) for the dual-slot A/B
  comparison workflow.

**Data flow**: Apps → OpenTelemetry Collector → ClickHouse (telemetry data) /
MongoDB (configuration/metadata)

## Development Setup (HOW)

```bash
yarn setup          # Install dependencies
yarn dev            # Start full stack with worktree-isolated ports
```

The project uses **Yarn 4.13.0** workspaces. Docker Compose manages ClickHouse,
MongoDB, and the OTel Collector.

**This repo is multi-agent friendly.** `yarn dev`, `make dev-int`, and
`make dev-e2e` all use slot-based port isolation so multiple worktrees can run
dev servers, integration tests, and E2E tests simultaneously without conflicts.
A dev portal at http://localhost:9900 auto-starts and shows all running stacks.
See [`agent_docs/development.md`](agent_docs/development.md) for the full
multi-worktree setup, port allocation tables, and available commands.

## Working on the Codebase (HOW)

**Before starting a task**, read relevant documentation from the `agent_docs/`
directory:

- `agent_docs/architecture.md` - Detailed architecture patterns and data models
- `agent_docs/tech_stack.md` - Technology stack details and component patterns
- `agent_docs/development.md` - Development workflows, testing, and common tasks
- `agent_docs/code_style.md` - Code patterns and best practices (read only when
  actively coding)
- `agent_docs/observability.md` - Instrumentation standards (tracing, metrics,
  context) and the shared helpers (read when adding or changing a feature)

**Package-specific guides** (read when working on that package):

- `packages/cli/AGENTS.md` - CLI/TUI architecture, keybindings, web frontend
  alignment, key patterns
- `packages/otel-collector/README.md` - Collector build process, included
  components, upgrade procedures, adding custom components
- `MCP.md` - MCP server setup and available tools (user-facing)

**After finishing all code edits**, run `yarn lint:fix` to auto-fix formatting
and lint issues across all packages. Pre-commit hooks handle this when
committing, but if you finish edits without committing, run `yarn lint:fix`
before stopping.

## Key Principles

1. **Multi-tenancy**: All data is scoped to `Team` - ensure proper filtering
2. **Type safety**: Use TypeScript strictly; Zod schemas for validation
3. **Existing patterns**: Follow established patterns in the codebase - explore
   similar files before implementing
4. **Component size**: Keep files under 300 lines; break down large components
5. **UI Components**: Use custom Button/ActionIcon variants (`primary`,
   `secondary`, `danger`) - see `agent_docs/code_style.md` for required patterns
6. **Testing**: Tests live in `__tests__/` directories; use Jest for
   unit/integration tests
7. **Observability**: This is an observability product - instrument new code as
   you write it. Every team-scoped operation must carry team/user context
   (`setBusinessContext`), and countable log events should also emit a metric.
   For our own instrumentation we favor wide events — enrich the unit-of-work
   span with rich, high-cardinality attributes and keep only span _names_ and
   _metric_ attributes low-cardinality — while metrics stay first-class
   (counters/histograms feed alerts and SLOs, and many deployments rely on
   them). Use the shared helpers in
   `packages/api/src/utils/instrumentation.ts`. See
   [`agent_docs/observability.md`](agent_docs/observability.md).

## Running Tests

Each package has different test commands available:

**packages/app** (unit tests only):

```bash
cd packages/app
yarn ci:unit           # Run unit tests
yarn dev:unit          # Watch mode for unit tests
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperdxio/hyperdx](https://github.com/hyperdxio/hyperdx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
