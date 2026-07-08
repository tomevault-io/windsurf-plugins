---
trigger: always_on
description: ├── package.json           # root (workspaces)
---

# ls-apis

## Project Structure

```
ls-apis/
├── package.json           # root (workspaces)
├── qa-output/             # QA reports (gitignored)
├── packages/
│   ├── aggregator/        # fetches, normalizes, deduplicates
│   │   ├── src/
│   │   │   ├── aggregate.ts       # main orchestration
│   │   │   ├── normalize.ts       # entry & category normalization
│   │   │   ├── paths.ts           # path utilities
│   │   │   ├── qa/                # QA validation
│   │   │   │   ├── index.ts       # QA orchestrator
│   │   │   │   ├── validations.ts # pure validation functions
│   │   │   │   └── tests/         # QA-specific tests
│   │   │   ├── sources/           # pluggable fetchers (*.fetcher.ts)
│   │   │   │   ├── index.ts       # fetcher auto-loader
│   │   │   │   └── tests/         # fetcher-specific tests
│   │   │   ├── tests/             # aggregator tests
│   │   │   └── types.ts           # ApiEntry, SourceFetcher interfaces
│   │   └── vitest.config.ts
│   ├── cli/               # CLI for searching APIs
│   │   ├── data/
│   │   │   └── apis.json          # bundled API data (published with package)
│   │   ├── src/
│   │   │   ├── index.ts           # CLI entry point
│   │   │   ├── paths.ts           # workspace root resolution
│   │   │   ├── colors.ts          # terminal color support
│   │   │   ├── qa.ts              # QA command handler
│   │   │   ├── categories.ts      # categories command
│   │   │   ├── providers.ts       # providers command
│   │   │   └── formatter.ts       # output formatter
│   │   └── tests/
│   │       ├── paths.test.ts      # path resolution tests
│   │       ├── qa.test.ts         # QA wrapper tests
│   │       ├── cli.test.ts        # CLI integration tests
│   │       ├── categories.test.ts # categories command tests
│   │       ├── providers.test.ts  # providers command tests
│   │       └── config.test.ts     # config tests
│   ├── shared/             # Shared types, config, search, paths (consumed by all packages)
│   │   ├── src/
│   │       ├── index.ts
│   │       ├── types.ts           # ApiEntry, DataFile, Provider, SearchOptions
│   │       ├── config.ts          # Config loading/display (moved from CLI)
│   │       ├── search.ts          # Search logic (moved from CLI)
│   │       └── paths.ts           # Workspace root resolution
│   └── mcp-server/         # MCP server for AI-friendly API queries (stdio transport)
│       └── src/
│           ├── index.ts           # Entry point
│           ├── server.ts          # MCP server with tools & resources
│           └── data.ts            # Data loading (apis.json)
└── AGENTS.md              # instructions for AI agents
```

## Commands

```bash
# Install deps
npm install

# Run aggregator (fetch all sources → dedupe → packages/cli/data/apis.json)
npm run aggregate

# Run QA checks on aggregated data (output to qa-output/issues.json)
npm run qa

# Run tests with coverage (both aggregator + cli)
npm test

# Run specific package tests
npm run test:aggregator
npm run test:cli

# Typecheck all workspaces
npm run typecheck

# MCP server (stdio transport for AI clients)
npm run mcp

# Lint & format
npm run lint
npm run lint:fix
npm run format
npm run format:fix

# CLI search (via npm script)
npm run ls-apis -- -q <query>
npm run ls-apis -- -c <category>
npm run ls-apis -- -a <auth>

# Build all packages (shared → CLI)
npm run build

# Build CLI only (requires shared to be built first)
npm run build --workspace=@ls-apis/cli

# CLI search (from compiled output, workspace root)
node packages/cli/dist/index.js -q <query>

# CLI search (globally, after npm link)
npm link --workspace=@ls-apis/cli
ls-apis -q <query>
```

## CLI Build Notes

- `@ls-apis/cli` publishes `dist/`.
- CLI package build uses `tsc` then `tsc-esm-fix --target dist`.
- `tsc-esm-fix` is required because Node ESM runtime requires explicit `.js` extensions in relative imports.
- CLI depends on `@ls-apis/shared` — shared must be built first (`npm run build` at root handles this).

## Shared Package Notes

- `@ls-apis/shared` uses `.js` extensions on all relative imports (required for Node.js ESM).
- Source `.ts` files import with `.js` extensions (e.g., `from './paths.js'`), which TypeScript resolves to `.ts` during compilation.
- `npm run build` at root builds shared first, then CLI.

## Architecture

- **Fetchers**: `*.fetcher.ts` files in `packages/aggregator/src/sources/`
- **Naming convention**: must end with `.fetcher.ts`
- **Interface**: implements `SourceFetcher` (name + sourceUrl + fetchApis())
- **Auto-loading**: via `loadAllFetchers()` in `sources/index.ts`
- **CLI colors**: `src/colors.ts` handles terminal coloring with chalk, respects `NO_COLOR` env and `--no-color` flag

## MCP Server Notes

- **MCP client config**: use `npx tsx packages/mcp-server/src/index.ts` for all platforms. VS Code will ask for permission once on first run — this is normal for project-local MCP servers (as opposed to published npm packages which are pre-trusted). Approving once persists the decision.
- **Auto-build**: `packages/mcp-server/index.js` is a small JS shim that builds the server (`tsc && tsc-esm-fix`) if `dist/` doesn't exist, then delegates to `dist/index.js`. This ensures MCP client configs work on fresh clones without a manual build step.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koalyptus/ls-apis](https://github.com/koalyptus/ls-apis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
