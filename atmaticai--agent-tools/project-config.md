---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Tools is an agent-driven data transformation platform for MCP (Model Context Protocol) and A2A (Agent-to-Agent) systems. It provides deterministic tools for transforming, formatting, and inspecting structured data across 18 tool domains: JSON, CSV, PDF, XML, Excel, Image, Markdown, Archive, Regex, Diff, SQL, Crypto, DateTime, Text, Math, Color, Physics, and Structural Engineering. The platform also includes an AI Chat interface that allows users to interact with all tools through natural language via HuggingFace-hosted LLMs.

**Philosophy**: "LLMs think. Agent Tools executes." - provides the authoritative execution layer for agents requiring strict correctness and repeatability.

## Commands

```bash
# Development
pnpm install              # Install dependencies
pnpm dev                  # Start all services (web + MCP server)
pnpm dev:web              # Start Next.js web app only (port 3000)
pnpm dev:mcp              # Start MCP server only

# Building
pnpm build                # Build all packages
pnpm build:web            # Build web app only
pnpm build:mcp            # Build MCP server only

# Testing
pnpm test                 # Run all tests (vitest)
pnpm --filter @atmaticai/agent-tools test           # Run core package tests only
pnpm --filter @atmaticai/agent-tools test     # Run MCP server tests only
cd packages/agent-tools && pnpm test:watch         # Watch mode for a specific package

# Code Quality
pnpm lint                 # ESLint across all packages
pnpm typecheck            # TypeScript type checking
pnpm format               # Prettier formatting
pnpm format:check         # Verify formatting
```

## Architecture

### Three-Layer Design

```
┌─────────────────────────────────────────────────────────────┐
│  UI Layer (apps/web)                                        │
│  Next.js 15 App Router, React 19, shadcn/ui, Zustand        │
├─────────────────────────────────────────────────────────────┤
│  packages/agent-tools                                       │
│  ├── MCP server (stdio/sse/http)                            │
│  ├── A2A agent (Agent-to-Agent protocol)                    │
│  └── apps/web/app/api/* (REST endpoints)                    │
├─────────────────────────────────────────────────────────────┤
│  Core Layer (packages/agent-tools/src/*)                    │
│  Business logic: json/, csv/, pdf/, xml/, excel/, image/,   │
│  markdown/, archive/, regex/, diff/, sql/, crypto/,         │
│  datetime/, text/, math/, color/, physics/, structural/     │
└─────────────────────────────────────────────────────────────┘
```

### Package Dependencies

- `@atmaticai/agent-tools` - Standalone, no internal dependencies
- `@atmaticai/agent-tools` - Depends on `@atmaticai/agent-tools`
- `@atmaticai/agent-tools/a2a` - Depends on `@atmaticai/agent-tools`
- `@atmaticai/agent-tools-web` - Depends on `@atmaticai/agent-tools` and `@atmaticai/agent-tools/a2a`

### Core Modules

| Module | Path | Key Dependencies |
|--------|------|-----------------|
| JSON | `packages/agent-tools/src/json/` | json5, yaml, smol-toml, jsonpath-plus, jmespath |
| CSV | `packages/agent-tools/src/csv/` | papaparse |
| PDF | `packages/agent-tools/src/pdf/` | pdf-lib, pdf-parse |
| XML | `packages/agent-tools/src/xml/` | fast-xml-parser |
| Excel | `packages/agent-tools/src/excel/` | exceljs |
| Image | `packages/agent-tools/src/image/` | sharp |
| Markdown | `packages/agent-tools/src/markdown/` | marked, turndown |
| Archive | `packages/agent-tools/src/archive/` | archiver, adm-zip |
| Regex | `packages/agent-tools/src/regex/` | (built-in) |
| Diff | `packages/agent-tools/src/diff/` | diff |
| SQL | `packages/agent-tools/src/sql/` | sql-formatter, node-sql-parser |
| Crypto | `packages/agent-tools/src/crypto/` | (Node.js crypto) |
| DateTime | `packages/agent-tools/src/datetime/` | luxon |
| Text | `packages/agent-tools/src/text/` | (built-in) |
| Math | `packages/agent-tools/src/math/` | (built-in) |
| Color | `packages/agent-tools/src/color/` | (built-in) |
| Physics | `packages/agent-tools/src/physics/` | (built-in) |
| Structural | `packages/agent-tools/src/structural/` | (built-in) |

### Core Module Structure

Each module in `packages/agent-tools/src/` follows a consistent pattern:
- `types.ts` - TypeScript types and interfaces
- `parse.ts` / `format.ts` - Input processing
- `transform.ts` - Data manipulation
- `validate.ts` - Schema/data validation
- `stats.ts` - Statistics extraction
- `index.ts` - Public exports (barrel file)

### Adding New Functionality

1. Implement in `packages/agent-tools/src/{module}/`
2. Add subpath export in `packages/agent-tools/package.json` and entry point in `tsup.config.ts`
3. Expose as MCP tool in `packages/agent-tools/src/tools/`
4. Expose as A2A skill in `packages/agent-tools/src/a2a/skills/`
5. Add REST route in `apps/web/app/api/{module}/`
6. Build UI page in `apps/web/app/(dashboard)/{module}/page.tsx`
7. Add sidebar navigation entry in `apps/web/components/layout/sidebar.tsx`

### Integration Patterns

**MCP Server** (for Claude Desktop, Claude Code):
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AtmaticAI/agent-tools](https://github.com/AtmaticAI/agent-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
