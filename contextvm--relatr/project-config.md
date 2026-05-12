---
trigger: always_on
description: [`relatr`](package.json) is a Bun/TypeScript service for computing personalized trust metrics for the Nostr ecosystem. It combines social graph traversal, profile validation, optional trusted assertions, and an MCP server interface. The default local entrypoint runs a browser-based configuration UI backed by [`manager.ts`](manager.ts).
---

# AGENTS.md

## Project Overview

[`relatr`](package.json) is a Bun/TypeScript service for computing personalized trust metrics for the Nostr ecosystem. It combines social graph traversal, profile validation, optional trusted assertions, and an MCP server interface. The default local entrypoint runs a browser-based configuration UI backed by [`manager.ts`](manager.ts).

### Main areas of the repository

- [`src/`](src): core service logic, MCP server, database, trust calculation, plugin runtime, and tests.
- [`config-ui/`](config-ui): browser UI served by the manager process for editing environment variables and controlling the service.
- [`plugins/elo/`](plugins/elo): portable plugin JSON events used by the Elo plugin engine.
- [`elo/`](elo): vendored sibling package for the Elo expression language used by the plugin system.

### Architecture notes

- The default runtime starts [`manager.ts`](manager.ts), which launches the config UI and then manages the compiled service process.
- The MCP server entrypoint is [`src/mcp/server.ts`](src/mcp/server.ts).
- Persistent storage uses DuckDB with schema file [`src/database/duckdb-schema.sql`](src/database/duckdb-schema.sql).
- The main service composition lives under [`src/service/`](src/service), with graph logic under [`src/graph/`](src/graph), capabilities under [`src/capabilities/`](src/capabilities), and plugin execution under [`src/plugins/`](src/plugins).

## Setup Commands

### Prerequisites

- Bun 1.x
- Docker and Docker Compose for containerized runs

### Install dependencies

```bash
bun install
```

### Initial environment setup

```bash
cp .env.example .env
openssl rand -hex 32
```

Set the generated key as `SERVER_SECRET_KEY` in [`.env`](.env.example). The canonical variable list and defaults live in [`.env.example`](.env.example).

## Development Workflow

### Default local development flow

Use the manager-based UI flow unless you specifically need the MCP server entrypoint:

```bash
bun start
```

This runs the [`start`](package.json:8) script from [`package.json`](package.json), which executes [`manager.ts`](manager.ts). The UI is intended to be available on port `3000` per [`README.md`](README.md:11).

### Run the MCP server directly

```bash
bun run mcp
```

This executes the [`mcp`](package.json:9) script and starts [`src/mcp/server.ts`](src/mcp/server.ts) directly.

### Useful maintenance commands

```bash
bun run lint
bun run typecheck
bun run format
bun run generate:version
bun run check:version
bun run build
bun run compile
```

- [`lint`](package.json:10): runs ESLint against [`src/`](src).
- [`typecheck`](package.json:11): runs the TypeScript compiler in no-emit mode to catch assignability and interface errors that ESLint alone will not report.
- [`format`](package.json:12): runs Prettier across the repository.
- [`generate:version`](package.json:13): regenerates [`src/version.ts`](src/version.ts) from [`package.json`](package.json) before builds.
- [`check:version`](package.json:14): regenerates [`src/version.ts`](src/version.ts) and fails if the tracked file would change, preventing version drift in CI.
- [`build`](package.json:15): builds the MCP server JS bundle into [`dist/`](dist).
- [`compile`](package.json:16): produces a standalone binary named [`relatr`](package.json:16).

### Docker workflow

The container flow is defined in [`docker-compose.yml`](docker-compose.yml) and [`Dockerfile`](Dockerfile).

```bash
cp .env.example .env
docker compose up -d
docker compose logs -f
docker compose down
```

Notes:

- Port `3000` is exposed by [`docker-compose.yml`](docker-compose.yml:31).
- Data is persisted in the local [`data/`](docker-compose.yml:29) bind mount.
- The image compiles both [`src/mcp/server.ts`](src/mcp/server.ts) and [`manager.ts`](manager.ts) during the Docker build in [`Dockerfile`](Dockerfile:24).

## Testing Instructions

### Root project tests

The root [`package.json`](package.json) currently does **not** define a dedicated [`test`](package.json:7) script. Existing root tests live in [`src/tests/`](src/tests) and use Bun's test runner.

Run all root tests with:

```bash
bun test src/tests
```

Run a single test file with:

```bash
bun test src/tests/plugin-management.test.ts
```

Target a specific test name with:

```bash
bun test src/tests -t "plugin"
```

### Elo subproject tests

The vendored Elo package has its own scripts in [`elo/package.json`](elo/package.json).

```bash
cd elo && bun install
cd elo && bun test test/unit/
```

Important: treat [`elo/`](elo) as a nested package with its own dependency graph and scripts. Root commands do not automatically cover it.

### Expectations for changes

- Add or update tests when changing behavior in [`src/`](src) or [`elo/src/`](elo/src).
- Run lint before finalizing changes.
- If you change build, plugin, MCP, database, or trust logic, run the most relevant Bun tests in [`src/tests/`](src/tests).

## Code Style Guidelines

### Language and tooling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ContextVM/relatr](https://github.com/ContextVM/relatr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
