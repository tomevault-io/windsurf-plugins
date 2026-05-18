---
trigger: always_on
description: This file provides guidance when working with code in this repository, including for Claude Code (claude.ai/code) and other AI coding assistants.
---

# Repository Guidelines

This file provides guidance when working with code in this repository, including for Claude Code (claude.ai/code) and other AI coding assistants.

## Documentation Philosophy

**IMPORTANT**: The documentation in `apps/docs/` describes the **intended system design and behavior**, not necessarily the current implementation state. Documentation represents the target architecture and how the system should work, serving as both specification and aspiration. When implementation and documentation differ, the documentation defines the goal, not a bug to be "fixed" in the docs.

## Overview

Studio is an open-source control plane for Model Context Protocol (MCP) traffic. It provides a unified layer for authentication, routing, and observability between MCP clients (Cursor, Claude, VS Code) and MCP servers. The system is built as a monorepo using Bun workspaces with TypeScript, Hono (API), and React 19 (UI).

## Commands

### Development
```bash
# Start full dev environment (migrations + client + server)
bun run dev

# Start mesh client only (Vite dev server on port 4000)
bun run --cwd=apps/mesh dev:client

# Start mesh server only (Hono with hot reload)
bun run --cwd=apps/mesh dev:server

# Run documentation site locally
bun run docs:dev
```

### Testing & Quality
```bash
# Run all tests (Bun test runner)
bun test

# Run tests for specific file/pattern
bun test path/to/file.test.ts

# TypeScript type checking (all workspaces)
bun run check

# Lint with oxlint and custom plugins
bun run lint

# Format code with Biome (ALWAYS run before committing)
bun run fmt

# Check formatting without modifying
bun run fmt:check
```

### Resilience Tests (Docker required)
```bash
# Run full resilience suite (builds containers, runs tests, tears down)
./tests/resilience/run.sh

# Or step by step:
docker compose -f tests/resilience/docker-compose.yml up -d --build --wait
bun test tests/resilience/scenarios/ --serial --timeout 900000
docker compose -f tests/resilience/docker-compose.yml down -v
```

Resilience tests use Docker Compose with Toxiproxy to simulate infrastructure failures (DB outages, NATS disconnections, high-latency MCP servers). See `tests/resilience/` for scenario files and configuration.

**IMPORTANT**: Always run `bun run fmt` after making code changes to ensure consistent formatting. A lefthook pre-commit hook is configured to run this automatically. Install with `npx lefthook install`.

### Database
```bash
# Run Kysely migrations (from apps/mesh/)
bun run --cwd=apps/mesh migrate

# Run Better Auth schema migrations
bun run --cwd=apps/mesh better-auth:migrate
```

#### Querying local postgres during development
The dev server uses embedded postgres on a **dynamic port**. To query it while `bun run dev` is running:

1. Find the port:
```bash
ps aux | grep "postgres -D" | grep -v grep
# Look for -p <PORT> at the end of the command
```

2. Run queries via a bun inline script (uses the `pg` package from apps/mesh):
```bash
cat << 'EOF' | bun run --cwd apps/mesh -
import pg from "pg";
const client = new pg.Client("postgresql://postgres:postgres@localhost:<PORT>/postgres");
await client.connect();
const { rows } = await client.query("SELECT * FROM <table> LIMIT 5");
console.log(JSON.stringify(rows, null, 2));
await client.end();
EOF
```

Replace `<PORT>` with the port found in step 1. The `--cwd apps/mesh` is required so bun resolves the `pg` dependency from the mesh workspace.

### Build & Deploy
```bash
# Build runtime package
bun run build:runtime

# Build mesh client (production)
bun run --cwd=apps/mesh build:client

# Build mesh server (bundle for deployment)
bun run --cwd=apps/mesh build:server

# Run production build
bun run --cwd=apps/mesh start
```

## Architecture

### Core Abstractions

**MeshContext** (`apps/mesh/src/core/mesh-context.ts`)
The central runtime interface injected into all tools. Provides:
- `auth`: Authentication state (user, session, organization)
- `access`: Access control layer (RBAC checks)
- `storage`: Database operations (Kysely-based)
- `vault`: Credential vault for secure token storage
- `tracer`: OpenTelemetry distributed tracing
- `meter`: OpenTelemetry metrics collection

Tools NEVER access HTTP objects, database drivers, or environment variables directly—all dependencies flow through MeshContext.

**defineTool()** (`apps/mesh/src/core/define-tool.ts`)
Declarative API for creating type-safe, auditable MCP tools. Automatically provides:
- Input/output validation (Zod schemas)
- Authorization checking (`ctx.access.check()`)
- Audit logging
- OpenTelemetry tracing and metrics
- Structured error handling

Example tool structure:
```typescript
export const EXAMPLE_TOOL = defineTool({
  name: "EXAMPLE_TOOL",
  description: "...",
  inputSchema: z.object({ ... }),
  outputSchema: z.object({ ... }),
  handler: async (input, ctx) => {
    await ctx.access.check(); // Authorization
    const result = await ctx.storage.someTable.create(...);
    return result;
  },
});
```

### Project Structure & Module Organization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [decocms/studio](https://github.com/decocms/studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
