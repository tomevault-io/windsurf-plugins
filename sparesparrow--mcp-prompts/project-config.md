---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Information

**Name:** @sparesparrow/mcp-prompts
**Version:** 3.14.0
**Description:** Cognitive development platform implementing MCP (Model Context Protocol) server for managing, versioning, and serving prompts with Claude Skills orchestration and self-improving AI-assisted development workflows

## Build & Development Commands

### Essential Commands
```bash
# Install dependencies
pnpm install

# Build the project (TypeScript + SWC)
pnpm run build

# Clean and rebuild
pnpm run build:clean

# Run tests
pnpm test                    # Run all tests once
pnpm run test:watch          # Watch mode
pnpm run test:coverage       # With coverage report

# Lint and type checking
pnpm run lint                # Check for issues
pnpm run lint:fix            # Auto-fix issues
pnpm run type-check          # TypeScript type checking only
```

### Development Modes
```bash
# Run in development with hot reload
pnpm run dev                 # Auto-detects mode from MODE env var
pnpm run dev:http            # HTTP REST API mode
pnpm run dev:mcp             # MCP stdio mode

# Run production build
pnpm start                   # Auto-detects mode
pnpm start:http             # HTTP mode
pnpm start:mcp              # MCP mode
```

### Docker Commands
```bash
# Build specific variants
pnpm run docker:build:mcp        # MCP stdio server
pnpm run docker:build:aws        # AWS-integrated variant
pnpm run docker:build:file       # File storage backend
pnpm run docker:build:postgres   # PostgreSQL backend
pnpm run docker:build:memory     # In-memory storage
pnpm run docker:build:all        # Build all variants

# Publishing
pnpm run publish:dry             # Dry-run publish to npm
pnpm run publish:npm             # Publish to npm registry
pnpm run publish:patch           # Bump patch version and publish
```

### AWS Deployment
```bash
# CDK deployment
pnpm run cdk:deploy          # Deploy all stacks
pnpm run cdk:destroy         # Tear down infrastructure

# Script-based deployment
./scripts/deploy-aws.sh      # Manual deployment
./scripts/cleanup-aws.sh     # Cleanup AWS resources
```

## Architecture Overview

### Hexagonal Architecture Pattern

The codebase follows **hexagonal architecture** (ports and adapters):

```
src/
├── core/                    # Domain layer (pure business logic)
│   ├── entities/           # Domain entities (Prompt, PromptMetadata)
│   ├── services/           # Business logic services
│   ├── ports/              # Interface definitions (contracts)
│   ├── events/             # Domain events
│   └── errors/             # Custom error types
│
├── adapters/               # Infrastructure implementations
│   ├── memory-adapter.ts   # In-memory storage
│   └── aws/                # AWS implementations
│       ├── dynamodb-adapter.ts
│       ├── s3-adapter.ts
│       └── sqs-adapter.ts
│
├── mcp/                    # MCP protocol implementation
│   ├── mcp-server.ts       # MCP server with tool registration
│   └── mcp-server.test.ts
│
├── lambda/                 # AWS Lambda handlers
│   ├── processor.ts
│   ├── catalog-sync.ts
│   ├── mcp-server.ts
│   └── stripe-webhook.ts
│
├── monitoring/             # CloudWatch metrics
├── index.ts                # HTTP server entry point
├── mcp-server-standalone.ts # MCP stdio entry point
├── cli.ts                  # CLI entry point
├── http-server.ts          # Express HTTP server
├── config.ts               # Configuration
└── schemas.ts              # Zod validation schemas
```

### Key Architectural Principles

1. **Domain Purity**: Core domain logic (`src/core/`) has no dependencies on adapters or infrastructure
2. **Dependency Injection**: All dependencies are injected via ports (interfaces)
3. **Port-Adapter Pattern**: Adapters implement port interfaces from `core/ports/`
4. **No Circular Dependencies**: Build order is core → adapters → apps/server

### Import Strategy

- Internal dependencies use `workspace:*` in package.json
- Import from built outputs: `@mcp-prompts/core/dist/`
- Use NodeNext module resolution
- Adapters can depend on `@mcp-prompts/core`
- Core NEVER imports from adapter packages

## Dual-Mode Server

This server runs in two distinct modes controlled by the `MODE` environment variable:

### MCP Mode (`MODE=mcp`)
- Stdio transport (standard input/output)
- Used by MCP clients like Claude Desktop
- Implements MCP 1.18 protocol specification
- Entry point: `src/mcp-server-standalone.ts`
- Tools exposed via `server.tool()` registration

### HTTP Mode (`MODE=http`)
- REST API with Express
- SSE support for streaming
- Entry point: `src/index.ts` → `src/http-server.ts`
- Endpoints: `/v1/prompts`, `/mcp/tools`, `/health`, etc.

## MCP Tools Implementation

MCP tools are registered using the `@modelcontextprotocol/sdk` and provide comprehensive prompt management capabilities:

```typescript
server.tool(
  "tool_name",
  "Description",
  zodSchema,
  async (args) => { /* implementation */ }
);
```

### Available MCP Tools

**Core Prompt Management:**
- **`list_prompts`** - Discover available prompts with filtering
  - Parameters: `cursor`, `tags[]`, `category`, `search`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sparesparrow/mcp-prompts](https://github.com/sparesparrow/mcp-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
