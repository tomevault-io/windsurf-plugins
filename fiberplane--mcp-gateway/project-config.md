---
trigger: always_on
description: This is a Bun workspace monorepo containing the MCP Gateway project. The repository structure follows monorepo best practices while maintaining backward compatibility for the main `@fiberplane/mcp-gateway` package.
---

# MCP Gateway Monorepo - Claude Code Instructions

## Project Overview

This is a Bun workspace monorepo containing the MCP Gateway project. The repository structure follows monorepo best practices while maintaining backward compatibility for the main `@fiberplane/mcp-gateway` package.

## Repository Structure

```
/Users/jaccoflenter/dev/fiberplane/mcp-gateway/
├── packages/
│   ├── mcp-gateway/             # @fiberplane/mcp-gateway (main package)
│   │   ├── src/                 # CLI orchestration
│   │   │   └── cli.ts           # CLI entry point
│   │   ├── bin/                 # Development CLI executable
│   │   ├── tests/               # Integration tests
│   │   ├── package.json         # CLI package configuration
│   │   └── tsconfig.json
│   ├── types/                   # @fiberplane/mcp-gateway-types
│   │   ├── src/                 # Type definitions and Zod schemas
│   │   ├── package.json         # Types package configuration
│   │   └── tsconfig.json
│   ├── core/                    # @fiberplane/mcp-gateway-core
│   │   ├── src/                 # Core business logic
│   │   │   ├── registry/        # Registry operations
│   │   │   ├── capture/         # MCP traffic capture
│   │   │   ├── logs/            # Log storage and queries
│   │   │   ├── utils/           # Shared utilities
│   │   │   ├── logger.ts        # Logging infrastructure
│   │   │   └── health.ts        # Health checks
│   │   ├── package.json         # Core package configuration
│   │   └── tsconfig.json
│   ├── management-mcp/          # @fiberplane/mcp-gateway-management-mcp
│   │   ├── src/                 # Gateway management MCP API
│   │   │   ├── tools/           # MCP tools (server & capture)
│   │   │   ├── app.ts           # MCP server creation
│   │   │   └── index.ts         # Public exports
│   │   ├── package.json         # Management MCP package configuration
│   │   ├── tsconfig.json
│   │   └── README.md            # Management MCP documentation
│   ├── server/                  # @fiberplane/mcp-gateway-server
│   │   ├── src/                 # HTTP server with proxy
│   │   │   ├── routes/          # Proxy and OAuth routes
│   │   │   ├── app.ts           # Hono application factory
│   │   │   └── index.ts         # Public exports
│   │   ├── package.json         # Server package configuration
│   │   ├── tsconfig.json
│   │   └── README.md            # Server documentation
│   ├── web/                     # @fiberplane/mcp-gateway-web
│   │   ├── src/                 # React web UI
│   │   │   ├── components/      # React components
│   │   │   ├── lib/             # API client and utilities
│   │   │   ├── App.tsx          # Main application
│   │   │   └── main.tsx         # Entry point
│   │   ├── public/              # Static assets (gitignored)
│   │   ├── package.json         # Web package configuration
│   │   ├── tsconfig.json
│   │   └── vite.config.ts       # Vite configuration
│   └── api/                     # @fiberplane/mcp-gateway-api
│       ├── src/                 # REST API for UI
│       │   ├── routes/          # API route handlers
│       │   ├── app.ts           # API app factory
│       │   └── index.ts         # Public exports
│       ├── package.json         # API package configuration
│       ├── tsconfig.json
│       └── README.md            # API documentation
├── test-mcp-server/             # Test MCP server for validation
│   ├── *.ts                     # Test server configurations
│   └── package.json             # Test server dependencies
├── scripts/                     # Shared build scripts
│   └── build.ts                 # Package build script
├── .github/workflows/           # CI/CD workflows
├── package.json                 # Root workspace configuration
└── [config files]              # Root-level configurations
```

## Important Commands

### Development Commands
- `bun install` - Install all workspace dependencies
- `bun run dev` - Start development mode (filters to CLI package)
- `bun run build` - Build all packages in dependency order (types → core → api → management-mcp → server → web → cli)
- `bun run clean` - Clean all dist folders
- `bun run test` - Run all tests across workspace (runs each workspace's tests with proper config)
- `bun run typecheck` - Type check all packages
- `bun run lint` - Lint all files with Biome
- `bun run format` - Format all files with Biome
- `bun run check-circular` - Check for circular dependencies (both within and between packages)
- `bun run deps-graph` - Generate dependency graph (deps.svg)

> **Shutdown Note**: `bun run dev` uses watch mode which intercepts SIGINT/SIGTERM. Shutdown messages/statistics may not appear and port 3333 may not release immediately. For clean shutdown, use `bun run --filter @fiberplane/mcp-gateway dev:no-watch` or wait ~1s before restarting.

### Package-Specific Commands
- `bun run --filter @fiberplane/mcp-gateway-types build` - Build types package
- `bun run --filter @fiberplane/mcp-gateway-core build` - Build core package
- `bun run --filter @fiberplane/mcp-gateway-api build` - Build API package

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fiberplane/mcp-gateway](https://github.com/fiberplane/mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
