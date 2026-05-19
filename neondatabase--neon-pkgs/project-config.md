---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a monorepo for Neon Launchpad packages that provide instant Postgres database provisioning without sign-ups. It contains five main packages:
- `get-db`: A CLI tool for creating claimable Neon databases (formerly `neondb`)
- `neondb`: Deprecated alias package for `get-db` (shows deprecation warning)
- `vite-plugin-db`: A Vite plugin that automatically provisions databases (formerly `@neondatabase/vite-plugin-postgres`)
- `@neondatabase/vite-plugin-postgres`: Deprecated alias package for `vite-plugin-db` (shows deprecation warning)
- `neon-init`: A setup tool for initializing your project with Neon and Neon MCP Server

## Development Commands

### Building

```bash
# Build both packages
pnpm build

# Build CLI only
pnpm build:cli

# Build Vite plugin only
pnpm build:plugin
```

### Testing

```bash
# Run all tests
pnpm test:ci

# Run tests for specific package
pnpm --filter get-db test
pnpm --filter vite-plugin-db test
```

### Linting & Formatting

```bash
# Format code (uses Biome)
pnpm format

# Run lint checks for CI
pnpm lint:ci
```

### Package Testing

```bash
# Test CLI with prompts
pnpm --filter get-db dry:run:prompt

# Test CLI with defaults
pnpm --filter get-db dry:run
```

## Architecture

### Monorepo Structure

-   Uses pnpm workspaces with two main packages
-   Shared dependencies managed at root level
-   Uses Biome for linting/formatting instead of ESLint/Prettier
-   Uses `tsdown` for TypeScript compilation instead of tsc directly
-   Package manager: pnpm@10.4.0, Node.js >=18.3.0
-   **Dependency Installation**: Prefer `pnpm dedupe` over `pnpm install` - it deduplicates dependencies in node_modules, minimizing conflict issues and reducing filesystem space

### get-db Package (formerly neondb)

-   **Entry points**: CLI (`dist/cli.js`) and SDK exports (`./sdk`, `./launchpad`)
-   **Core functionality**: Creates claimable Neon databases via API calls
-   **CLI options**: `-y/--yes`, `-e/--env`, `-k/--key`, `-s/--seed`, `-h/--help`
-   **Dependencies**: Uses `@clack/prompts` for interactive CLI, `@neondatabase/serverless` for DB operations

### neondb Package (DEPRECATED)

-   **Purpose**: Deprecated alias for `get-db` - shows deprecation warning and re-exports `get-db`
-   **Status**: Maintained for backwards compatibility but users should migrate to `get-db`

### vite-plugin-db Package (formerly @neondatabase/vite-plugin-postgres)

-   **Purpose**: Automatically provisions databases during Vite development
-   **Behavior**: Checks for DATABASE_URL in .env, creates database if missing, noop in production
-   **Configuration**: Supports custom env file path, env key name, and SQL seeding
-   **Integration**: Must be placed as first plugin in Vite config
-   **Dependencies**: Uses `get-db` internally for database provisioning

### @neondatabase/vite-plugin-postgres Package (DEPRECATED)

-   **Purpose**: Deprecated alias for `vite-plugin-db` - shows deprecation warning and re-exports `vite-plugin-db`
-   **Status**: Maintained for backwards compatibility but users should migrate to `vite-plugin-db`

### neon-init Package

-   **Purpose**: Setup tool for configuring Neon with the user's project
-   **Entry points**: CLI (`dist/cli.js`) and SDK export (`init()` function)
-   **Core functionality**: Installs Neon Local Connect extension for VS Code/Cursor, configures MCP server for Claude CLI, and installs Neon agent skills
-   **Behavior**:
    -   Detects available editors installed on the system
    -   Prompts user to select which editor(s) to configure
    -   Authenticates via OAuth using `neonctl`
    -   Creates API key automatically via Neon API
    -   For **VS Code** and **Cursor**: Installs Neon Local Connect extension (`databricks.neon-local-connect`) via CLI commands, configures API key via URI handler
    -   For **Claude CLI**: Writes MCP configuration to `~/.claude.json`
    -   Installs Neon agent skills via Vercel's `skills` CLI (runs `npx skills add neondatabase/agent-skills`)
-   **Extension Installation**:
    -   Uses `code --install-extension` or `cursor --install-extension` commands
    -   Finds CLI paths by checking known installation locations and using `mdfind` on macOS
    -   Waits for extension to be fully installed before configuring
    -   Configures API key via URI handler: `vscode://databricks.neon-local-connect/import-api-key?token=xxx`
-   **MCP Server Configuration**:
    -   Only used for Claude CLI (not VS Code/Cursor)
    -   Writes to `~/.claude.json` with remote MCP server URL and API key
-   **Agent Skills Integration**:
    -   Maps editors to agent names: Cursor → `cursor`, VS Code → `github-copilot`, Claude CLI → `claude-code`
    -   Runs skills installation with `-y` (auto-confirm) flag
    -   Handles failures gracefully per-editor, continues with remaining editors
-   **Dependencies**: Uses `@clack/prompts` for interactive CLI, `execa` for running external commands (`neonctl`, extension CLI commands, `npx`, URI handlers)

### Key Implementation Details

-   Both packages support SQL seeding via `--seed` flag (CLI) or `seed.path` option (plugin)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neondatabase/neon-pkgs](https://github.com/neondatabase/neon-pkgs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
