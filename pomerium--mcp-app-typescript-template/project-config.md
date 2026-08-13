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

npm workspaces split the codebase: `server/` is the MCP backend and `widgets/` houses React widgets.

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

This template uses `McpServer` from `@modelcontextprotocol/server` and `createMcpHandler` with the MCP Apps helpers:

- Register UI resources with `registerAppResource`
- Register tools with `registerAppTool`
- Include `_meta.ui.resourceUri` on tools to bind a UI resource

### Stateless HTTP

The `/mcp` endpoint follows the MCP 2026-07-28 stateless transport model:

- `createMcpHandler` creates a fresh server and transport for every request
- Modern requests carry protocol version, client identity, and capabilities in `_meta`
- The server does not create, store, or return `Mcp-Session-Id` values
- Older 2025-era requests are served through the SDK's per-request stateless fallback
- State that must survive a request must be represented explicitly in tool arguments or an external store

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
import { useEffect, useState } from 'react';

function MyWidget() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pomerium/mcp-app-typescript-template](https://github.com/pomerium/mcp-app-typescript-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
