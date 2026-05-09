---
trigger: always_on
description: This file provides guidance to AI assistants (including Claude Code at claude.ai/code) when working with code in this repository.
---

# Repository Guidelines

This file provides guidance to AI assistants (including Claude Code at claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP Apps template built with the MCP Apps spec and Model Context Protocol (MCP). The architecture consists of:

- **MCP Server** (Node.js + Express): Handles tool registration, execution, and widget resource serving
- **MCP App Views**: Interactive components rendered in host iframes that communicate via the MCP Apps `App` API
- **Widget Build System**: Custom Vite-based parallel build pipeline with content hashing and auto-discovery

npm workspaces split the codebase: `server/` is the MCP backend, `widgets/` houses React widgets, and shared tooling sits in `scripts/`.

## Development Commands

### Quick Start

**Fastest way to get up and running:**

```bash
npm install
npm run dev
```

This starts both the MCP server (`http://localhost:8080`) and widget dev server (`http://localhost:4444`).

### All Available Commands

**Development:**

```bash
npm run dev           # Start everything (server + widgets in watch mode)
npm run dev:inline    # Inlined assets for Claude.ai or remote sharing via ssh -R 0 pom.run
npm run dev:server    # Start only MCP server (watch mode)
npm run dev:widgets   # Start only widget dev server
npm run inspect       # Test with MCP Inspector
```

**Building:**

```bash
npm run build         # Full production build (widgets + server)
npm run build:widgets # Build only widgets
npm run build:server  # Build only server
```

**Testing:**

```bash
npm test              # Run all tests
npm run test:server   # Run server tests only
npm run test:widgets  # Run widget tests only
npm run test:coverage # Run tests with coverage
```

**Code Quality:**

```bash
npm run lint          # Lint TypeScript files
npm run format        # Format code with Prettier
npm run format:check  # Check formatting without modifying
npm run type-check    # Type check all workspaces
```

**Storybook:**

```bash
npm run storybook        # Run Storybook dev server
npm run build:storybook  # Build Storybook for production
```

## Key Architectural Patterns

### MCP Apps Server Usage

This template uses `McpServer` from `@modelcontextprotocol/sdk/server/mcp.js` with the MCP Apps helpers:

- Register UI resources with `registerAppResource`
- Register tools with `registerAppTool`
- Include `_meta.ui.resourceUri` on tools to bind a UI resource

### Widget Resource Registration

Widgets MUST be registered with the exact MIME type `text/html;profile=mcp-app` for MCP Apps hosts to load them:

```typescript
registerAppResource(
  server,
  'ui://my-widget',
  'ui://my-widget',
  { mimeType: 'text/html;profile=mcp-app' }, // CRITICAL - must be exact
  async () => ({
    contents: [
      {
        uri: 'ui://my-widget',
        mimeType: 'text/html;profile=mcp-app',
        text: html,
      },
    ],
  })
);
```

### Tool Response Structure

All tool responses follow this pattern (UI binding happens in tool metadata):

```typescript
{
  content: [{ type: 'text', text: 'Human-readable message' }],
  structuredContent: {
    // Data passed to the app via App.ontoolresult
    // Keep this under 4,000 tokens for performance
  },
  // No outputTemplate required; UI linkage lives in tool _meta.ui.resourceUri
}
```

### Session Management

The server uses `SessionManager` (server/src/utils/session.ts) to track MCP sessions:

- Sessions are created per HttpStreamable connection with unique IDs
- Session IDs are communicated via the `mcp-session-id` header
- Automatic cleanup of stale sessions runs based on `SESSION_MAX_AGE` (default 1 hour)
- Each session has its own MCP server instance to maintain isolation
- Session data includes server instance, transport, and creation timestamp
- Resumability is enabled via `InMemoryEventStore` for handling connection interruptions

### Widget Build System

Vite auto-discovers and builds widgets via a custom plugin:

- Scans `widgets/src/widgets/*.{tsx,jsx}` for widget entry points
- Widget name comes from the filename (e.g., `echo.tsx` → `echo` widget)
- **Widgets must include their own mounting code** at the bottom of the file
- Generates content-hashed assets (e.g., `echo-a1b2c3d4.js`)
- Creates HTML templates with preload hints that reference hashed assets
- Both hashed and unhashed filenames are generated for flexibility
- Widget bundles in `assets/` are generated artifacts; never edit them manually

**Widget folder structure:**

```
widgets/src/
  ├── widgets/              # Widget entry points (auto-discovered)
  │   ├── echo.tsx          # Widget entry - includes mounting code
  │   └── counter.tsx       # Another widget entry
  ├── echo/                 # Widget-specific components
  │   ├── Echo.tsx
  │   └── Echo.stories.tsx
  ├── components/           # Shared components (including shadcn/ui)
  │   └── ui/
  ├── hooks/                # Shared hooks
  └── utils/                # Shared utilities
```

**To add a new widget:**

1. Create `widgets/src/widgets/my-widget.tsx`:

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { App } from '@modelcontextprotocol/ext-apps';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pomerium/chatgpt-app-typescript-template](https://github.com/pomerium/chatgpt-app-typescript-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
