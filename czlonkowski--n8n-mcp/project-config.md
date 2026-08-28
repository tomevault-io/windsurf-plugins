---
trigger: always_on
description: > This file is committed to a public OSS repository. Never add API keys, credentials, private URLs, customer data, or private infrastructure details.
---

# Repository Guidelines

> This file is committed to a public OSS repository. Never add API keys, credentials, private URLs, customer data, or private infrastructure details.

## Project Overview

n8n-mcp is a Model Context Protocol server that gives AI assistants access to n8n node documentation, workflow validation, and workflow management. Documentation and validation tools work offline against the bundled SQLite node database. Management tools prefixed with `n8n_` operate on a live n8n instance and require API configuration.

## Project Structure & Module Organization

Core TypeScript lives in `src/`. Important subsystems include:

- `src/mcp/` — MCP server, request handlers, tool definitions and documentation, and bundled skills
- `src/database/` — SQLite adapters, repositories, FTS5 search, and migrations
- `src/loaders/`, `src/parsers/`, `src/mappers/` — node loading, metadata parsing, and documentation mapping
- `src/services/` — validation, workflow diffing and autofix, node/version lookup, the n8n API client, and security scanners
- `src/templates/` and `src/community/` — workflow-template and community-node ingestion and documentation
- `src/telemetry/`, `src/triggers/`, and `src/n8n/` — telemetry, trigger detection, and the n8n community-node wrapper
- `src/scripts/` — maintenance scripts compiled to `dist/scripts/`
- `src/types/`, `src/constants/`, and `src/utils/` — shared types, constants, and helpers
- `src/http-server*.ts` — HTTP transports and session persistence
- `src/mcp-engine.ts` and `src/mcp-tools-engine.ts` — APIs for embedding the server

Tests mirror the source areas in `tests/unit/` and `tests/integration/`, with fixtures, factories, helpers, and mocks under `tests/`. React/Vite apps live in `ui-apps/src/`, repository utilities in `scripts/`, documentation in `docs/`, and generated skills and databases in `data/`. Treat `dist/`, coverage output, and `ui-apps/dist/` as generated; do not edit them directly.

## Architecture & MCP Conventions

- Route database operations through repository classes and keep business logic in the service layer.
- Validation profiles are `minimal`, `runtime`, `ai-friendly`, and `strict`.
- Prefer diff-based workflow changes through `n8n_update_partial_workflow`; do not replace a whole workflow when a focused operation is sufficient.
- Offline documentation and validation tools include `search_nodes`, `get_node`, `validate_node`, `validate_workflow`, `search_templates`, `get_template`, and `tools_documentation`.
- Live management tools use the `n8n_*` prefix and cover workflows, executions, tests, versions, autofix, templates, credentials, datatables, and audits.
- Request the smallest useful `get_node` detail level: `minimal`, `standard`, or `full`.
- Validate workflows before deploying them to n8n.

## Build, Test, and Development Commands

```bash
# Install and build
npm install                    # Install root dependencies; repeat in ui-apps/ for UI work
npm run build                  # Compile production TypeScript to dist/
npm run build:all              # Sync skills, build UI apps, and compile the server
npm run typecheck              # Strict TypeScript check without emitting; npm run lint is an alias

# Test
npm test                       # Run all Vitest tests
npm run test:unit              # Unit tests
npm run test:integration       # Integration tests
npm run test:e2e               # End-to-end tests
npm run test:coverage          # Coverage report
npm test -- tests/unit/services/property-filter.test.ts

# Run and maintain
npm start                      # MCP server in stdio mode
npm run start:http             # MCP server in HTTP mode
npm run dev:http               # Rebuild and restart HTTP mode on source changes
npm run rebuild                # Rebuild the bundled node database
npm run validate               # Validate generated node data
npm run dev                    # Build, rebuild the database, and validate

# Update bundled data
npm run update:n8n:check       # Dry-run n8n dependency update; follow MEMORY_N8N_UPDATE.md
npm run update:n8n             # Update n8n packages
npm run fetch:templates        # Fetch n8n.io templates; follow MEMORY_TEMPLATE_UPDATE.md
npm run fetch:community        # Upsert community nodes while preserving existing docs
npm run generate:docs:incremental # Generate docs for community nodes missing them
```

Database rebuilds take several minutes because of the n8n package size. HTTP mode requires valid auth configuration, and live n8n tests require configuration and a clean database state.

## Coding Style & Naming Conventions

Use strict TypeScript, two-space indentation, single quotes, and semicolons. Prefer `camelCase` for variables and functions, `PascalCase` for classes and types, and kebab-case filenames such as `workflow-auto-fixer.ts`. Use the configured `@/` and `@tests/` aliases where helpful. Keep modules focused, validate external input, and do not use hyperbolic or dramatic language in comments or documentation. No separate formatter is configured; `npm run typecheck` is the required static check.

## Development & Testing Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czlonkowski/n8n-mcp](https://github.com/czlonkowski/n8n-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
