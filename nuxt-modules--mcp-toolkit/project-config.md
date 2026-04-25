---
trigger: always_on
description: A guide for AI coding agents working on the Nuxt MCP Toolkit project.
---

# AGENTS.md

A guide for AI coding agents working on the Nuxt MCP Toolkit project.

## Project Overview

**Nuxt MCP Toolkit** is a Nuxt module that enables developers to create [Model Context Protocol (MCP)](https://modelcontextprotocol.io) servers directly in their Nuxt applications. It provides automatic discovery of tools, resources, and prompts with zero configuration - just create files and they're automatically registered.

### Monorepo Structure

This is a pnpm monorepo managed with Turborepo:

```
nuxt-mcp-toolkit/
├── packages/
│   └── nuxt-mcp-toolkit/     # Main module (published as @nuxtjs/mcp-toolkit)
├── apps/
│   ├── docs/                 # Documentation site (mcp-toolkit.nuxt.dev)
│   ├── playground/           # Development playground for testing
│   └── mcp-starter/        # Minimal MCP template (`pnpm dev:starter`)
```

## Development Environment Setup

### Prerequisites

- Node.js 18+
- pnpm 9.15.0+

### Initial Setup

```bash
# Install dependencies
pnpm install

# Generate type stubs (required before first run)
pnpm run dev:prepare

# Start the playground
pnpm run dev

# Start the docs site
pnpm run dev:docs

# Start the minimal MCP starter
pnpm run dev:starter
```

## Common Commands

Run from the repository root:

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start the playground app |
| `pnpm dev:starter` | Start the minimal MCP starter app |
| `pnpm dev:docs` | Start the documentation site |
| `pnpm build` | Build all packages |
| `pnpm build:module` | Build only the module |
| `pnpm build:docs` | Build only the docs |
| `pnpm test` | Run all tests |
| `pnpm lint` | Run ESLint |
| `pnpm lint:fix` | Fix ESLint issues |
| `pnpm typecheck` | Run TypeScript type checking |
| `pnpm eval` | Run MCP evals (docs) |
| `pnpm eval:ui` | Run MCP evals with UI (docs) |

## Project Structure

### Main Module (`packages/nuxt-mcp-toolkit/`)

```
packages/nuxt-mcp-toolkit/
├── src/
│   ├── module.ts                    # Main module entry point
│   └── runtime/
│       ├── components/              # Vue components (InstallButton)
│       └── server/
│           ├── mcp/
│           │   ├── definitions/     # Tool, resource, prompt definitions
│           │   ├── loaders/         # File discovery and loading
│           │   ├── validators/      # Zod validation logic
│           │   ├── handler.ts       # MCP HTTP handler
│           │   └── utils.ts         # Utility functions
│           └── types/               # TypeScript types
└── test/
    ├── *.test.ts                    # Test files
    ├── fixtures/                    # Test fixtures (mini Nuxt apps)
    └── helpers/                     # Test utilities
```

### Documentation (`apps/docs/`)

Built with Nuxt Content. MCP definitions are in `server/mcp/`:

```
apps/docs/server/mcp/
├── tools/           # MCP tools (list-pages, get-page)
├── prompts/         # MCP prompts (create-tool, troubleshoot, etc.)
└── resources/       # MCP resources
```

### Playground (`apps/playground/`)

A full-featured example app demonstrating module usage with authentication, todos, and various MCP definitions.

### MCP Starter (`apps/mcp-starter/`)

A minimal Nuxt app with one tool, one resource, and one prompt (explicit `@nuxtjs/mcp-toolkit/server` imports). Readers scaffold **only** this folder via giget/tiged (see [apps/mcp-starter/README.md](apps/mcp-starter/README.md)). Short blog paste: [PROMPT.md](apps/mcp-starter/PROMPT.md). In the monorepo, run **`pnpm build:module`** before `pnpm dev:starter` so `server` exports exist.

## Code Style and Conventions

### General

- **TypeScript** is required for all code
- **ESLint** with `@nuxt/eslint-config` (stylistic rules enabled)
- **Zod** for schema validation (use `z` from `zod`)
- Run `pnpm lint:fix` before committing

### MCP Definitions

Use the helper functions:

```typescript
// Tools - server/mcp/tools/*.ts (or subdirectories like tools/admin/*.ts)
import { z } from 'zod'
import { defineMcpTool } from '@nuxtjs/mcp-toolkit/server'

export default defineMcpTool({
  name: 'tool-name',           // Optional - auto-generated from filename
  group: 'admin',              // Optional - auto-inferred from subdirectory
  tags: ['destructive'],       // Optional - free-form tags for filtering
  description: 'What it does',
  inputSchema: {
    param: z.string().describe('Parameter description'),
  },
  handler: async ({ param }) => {
    return 'Result' // string, number, boolean, object, or full CallToolResult
  },
})

// Resources - server/mcp/resources/*.ts
import { defineMcpResource } from '@nuxtjs/mcp-toolkit/server'

export default defineMcpResource({
  name: 'resource-name',
  uri: 'file:///path/or/pattern',
  handler: async (uri: URL) => {
    return {
      contents: [{ uri: uri.toString(), text: 'Content' }],
    }
  },
})

// Prompts - server/mcp/prompts/*.ts
import { z } from 'zod'
import { defineMcpPrompt } from '@nuxtjs/mcp-toolkit/server'

export default defineMcpPrompt({
  name: 'prompt-name',
  inputSchema: {
    arg: z.string(),
  },
  handler: async ({ arg }) => {
    return {
      messages: [{
        role: 'user',
        content: { type: 'text', text: `Message with ${arg}` },
      }],
    }
  },
})
```

### Auto-Generated Names


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuxt-modules/mcp-toolkit](https://github.com/nuxt-modules/mcp-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
