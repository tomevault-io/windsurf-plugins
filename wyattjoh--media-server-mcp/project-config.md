---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Type checking - Always run before committing
deno check

# Linting - Always run before committing  
deno lint

# Code formatting
deno fmt

# Run tests
deno test --allow-net

# Development with hot reload
deno task dev

# Development with debug logging enabled
deno task dev --debug

# Production run
deno task start

# Production run with debug logging enabled  
deno task start --debug

# SSE mode with debug logging
deno task dev:sse --debug
deno task start:sse --debug

# Streamable HTTP mode (recommended for remote MCP)
deno task dev:http --debug
deno task start:http --debug

# Streamable HTTP mode with custom host/port
deno task start:http --port 8080 --host 127.0.0.1
```

## Monorepo Structure

This repository is organized as a Deno workspace with the following packages:

- **packages/radarr/** - `@wyattjoh/radarr` - Radarr API client library
- **packages/sonarr/** - `@wyattjoh/sonarr` - Sonarr API client library
- **packages/tmdb/** - `@wyattjoh/tmdb` - TMDB API client library
- **packages/plex/** - `@wyattjoh/plex` - Plex API client library
- **packages/media-server-mcp/** - `@wyattjoh/media-server-mcp` - Main MCP server

Each package is independently publishable and has its own `deno.json` configuration.

## Development Best Practices

- Always use `deno task fmt`, `deno task lint`, and `deno task check` after modifying or creating code to ensure that it's correct.
- Run `deno test --allow-net` to verify all tests pass before committing changes.
- Tests are organized by layer: `packages/media-server-mcp/tests/` contains `tools/` (tool tests), `server_test.ts`, `auth_test.ts`, and transport tests (`sse-transport_test.ts`, `streamable-http-transport_test.ts`). Each client package also has its own `tests/` directory.
- After changing any of the available MCP tools or resources, evaluate if you need to update the README.md and CLAUDE.md to be reflective of those changes.
- When creating pull requests, always use the PR template at `.github/pull_request_template.md`.

### File Naming Conventions

- **Source files**: Use kebab-case for all source files (e.g., `query-enhancer.ts`, `search-service.ts`)
- **Test files**: Use kebab-case with `_test.ts` suffix (e.g., `query-enhancer_test.ts`, `search-service_test.ts`)
- **Directory structure**: Tests mirror the source structure in the `packages/media-server-mcp/tests/` directory

## Architecture Overview

This is a **Model Context Protocol (MCP) server** that provides AI assistants with tools, resources, and prompts to manage Radarr (movies), Sonarr (TV series), and Plex media servers, and access TMDB data through their APIs.

### Core Architecture Pattern

The codebase follows a **layered architecture**:

1. **MCP Server Layer** (`packages/media-server-mcp/src/index.ts`): Main server that handles MCP protocol communication
2. **Tool Layer** (`packages/media-server-mcp/src/tools/`): MCP tool definitions and handlers that bridge MCP and API clients
3. **Resource Layer** (`packages/media-server-mcp/src/resources/`): MCP resources that expose service configs and dynamic data as readable URIs
4. **Prompt Layer** (`packages/media-server-mcp/src/prompts/`): MCP prompts for common media management workflows
5. **Client Packages** (`packages/{radarr,sonarr,tmdb,plex}/`): Standalone client libraries for each service
6. **Type Definitions**: Each package contains its own TypeScript definitions
7. **Shared Components**: Client packages include filtering and validation utilities

### Key Architectural Decisions

**Direct Tool Registration**: Tools are registered directly on the `McpServer` instance via `createRadarrTools()`, `createSonarrTools()`, `createTMDBTools()`, and `createPlexTools()` functions. Each function accepts the server, service config, and a tool filter function, then registers tools as side effects using `server.registerTool()`. Tool handlers are closures that capture the service config at registration time. All tool handlers are wrapped with `wrapToolHandler()` from `tool-wrapper.ts`, which centralizes error handling (returning `isError: true` on failure), logging, and execution timing. Individual tool handlers do not need their own try/catch blocks.

**Tool Filtering System**: A configurable tool filtering system (`tool-categories.ts`, `tool-filter.ts`) allows enabling/disabling tools via profiles, branches, include/exclude lists, or config files. This controls which tools are registered on the server.

**Configuration Injection**: The main server maintains optional service configurations (`radarrConfig`, `sonarrConfig`, `tmdbConfig`, `plexConfig`) and passes them into tool registration functions, allowing the server to work with any combination of services configured.

**Environment-Based Configuration**: Service availability is determined by environment variables at startup. Missing configuration results in that service's tools being unavailable rather than failing the entire server.

### Type System

- **Strict TypeScript**: Uses `exactOptionalPropertyTypes: true` - optional properties must be explicitly `| undefined` rather than using `?:`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyattjoh/media-server-mcp](https://github.com/wyattjoh/media-server-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
