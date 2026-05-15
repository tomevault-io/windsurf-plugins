---
trigger: always_on
description: This is a TypeScript-based MCP (Model Context Protocol) server that enables AI agents to interact with Gremlin-compatible graph databases. The server provides 6 tools and 2 resources for comprehensive graph database operations.
---

## Project Overview

This is a TypeScript-based MCP (Model Context Protocol) server that enables AI agents to interact with Gremlin-compatible graph databases. The server provides 6 tools and 2 resources for comprehensive graph database operations.

**Implementation Status**: This project is fully implemented using **Effect.ts** functional programming patterns throughout.

## Architecture

### Core Components

- **server.ts** - Effect-based MCP server with graceful startup/shutdown using Effect composition
- **gremlin/service.ts** - Effect-based Gremlin service with Context.Tag service pattern
- **gremlin/schema-service.ts** - Effect-based schema management with dependency injection
- **config.ts** - Effect.Config-based configuration validation (no backward compatibility)
- **handlers/** - Effect-based modular MCP request handlers with service injection
- **utils/** - Effect-based utility modules with composable operations

### Available Tools

1. `get_graph_status` - Check database connection status
2. `get_graph_schema` - Get complete schema with vertex/edge labels and relationship patterns
3. `run_gremlin_query` - Execute any Gremlin traversal query
4. `refresh_schema_cache` - Force immediate refresh of cached schema
5. `import_graph_data` - Import data from GraphSON or CSV formats with batch processing
6. `export_subgraph` - Export subgraph data to JSON, GraphSON, or CSV formats

### Available Resources

1. `gremlin://status` - Real-time connection status
2. `gremlin://schema` - Cached schema information (JSON format)

## Development Rules

### MUST Follow RULES.md

Always follow the development rules in `RULES.md` - these are mandatory for all code changes, testing, security, and architecture decisions.

### Code Standards

- Use TypeScript with strict type checking
- Follow ESLint and Prettier configurations
- Use Effect.ts functional programming patterns throughout
- Use Zod for runtime validation with Effect.Config
- Implement Effect-based error handling with custom error types
- Write tests for all new functionality using Effect testing patterns

### Testing Requirements

- Use Jest with ts-jest for ESM support
- Mock external dependencies properly
- Include integration tests for end-to-end validation
- Maintain high test coverage

### Key Patterns

- **Effect.ts Functional Programming**: All operations use Effect's composable, type-safe approach
- **Service-Oriented Architecture**: Dependencies managed through Effect's Context.Tag patterns
- **Layer-Based Composition**: Application built using Effect.Layer for dependency resolution
- **Immutable State Management**: All state changes handled through Effect.Ref
- **Type Safety**: Comprehensive TypeScript with Effect's type system + Zod validation
- **Configuration Management**: Effect.Config-based environment validation
- **Error Handling**: Effect-based error handling with custom error types
- **Generic Gremlin Support**: Compatible with any Gremlin database

## Development Commands

### Essential Commands

```bash
npm run validate    # Run complete validation (format, lint, type-check, test)
npm run dev        # Development mode with hot reload
npm test          # Run all tests
npm run build     # Compile TypeScript to JavaScript
```

### Testing Commands

```bash
npm run test:watch     # Run tests in watch mode
npm run test:coverage  # Generate test coverage report
npm run test:it       # Run integration tests (requires GREMLIN_ENDPOINT)
```

### Code Quality Commands

```bash
npm run lint          # Run ESLint on source and test files
npm run lint:fix      # Auto-fix ESLint issues
npm run format        # Format code with Prettier
npm run format:check  # Check code formatting
```

## Environment Variables

Required:

- `GREMLIN_ENDPOINT` - Server endpoint (host:port or host:port/traversal_source)

Optional:

- `GREMLIN_USE_SSL` - Enable SSL/TLS connections (default: false)
- `GREMLIN_USERNAME` - Authentication username
- `GREMLIN_PASSWORD` - Authentication password
- `GREMLIN_IDLE_TIMEOUT` - Connection idle timeout in seconds (default: 300)
- `LOG_LEVEL` - Logging level: error, warn, info, debug (default: info)

## File Structure Guidelines

### Source Code (`src/`)

- **server.ts** - Effect-based MCP server with graceful startup/shutdown
- **config.ts** - Effect.Config-based configuration validation (no backward compatibility)
- **constants.ts** - Application constants integrated with Effect configuration
- **gremlin/** - Effect-based graph database code
  - **service.ts** - GremlinService using Effect.Context.Tag pattern
  - **schema-service.ts** - SchemaService with Effect dependency injection
  - **models.ts** - Zod schemas and TypeScript types
- **handlers/** - Effect-based MCP request handlers
  - **resources.ts** - Effect-based resource handlers with service injection
  - **tools.ts** - Effect-based tool handlers with createEffectTool helpers
  - **runtime-context.ts** - ManagedRuntime container for Effect execution
- **utils/** - Effect-based utility modules
  - **type-guards.ts** - Runtime type checking functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kpritam/gremlin-mcp](https://github.com/kpritam/gremlin-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
