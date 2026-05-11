---
trigger: always_on
description: Welcome! This document helps you navigate the WebMCP Examples repository efficiently.
---

## FOR AI AGENTS

Welcome! This document helps you navigate the WebMCP Examples repository efficiently.

**Start here:** [CONTRIBUTING.md](./CONTRIBUTING.md) - Development standards and best practices

## Quick Navigation

### Project Overview
- **[README.md](./README.md)** - What these examples demonstrate, quick start guide
- **[CONTRIBUTING.md](./CONTRIBUTING.md)** - How to contribute new examples or improve existing ones

### Current Examples (Modern WebMCP API)

#### Vanilla JavaScript/TypeScript
- **[vanilla/README.md](./vanilla/README.md)** - Shopping cart example using `@mcp-b/global`
- **Location**: `/vanilla`
- **Key file**: `vanilla/src/main.ts` - Entry point with WebMCP tool registration
- **API used**: `navigator.modelContext.registerTool()`

#### React + TypeScript
- **[react/README.md](./react/README.md)** - Task manager using `@mcp-b/react-webmcp`
- **Location**: `/react`
- **Key file**: `react/src/App.tsx` - React component with `useWebMCP` hooks
- **API used**: `useWebMCP()` hook

#### Rails + Stimulus
- **[rails/README.md](./rails/README.md)** - Bookmarks manager using Stimulus controllers
- **Location**: `/rails`
- **Key file**: `rails/app/javascript/controllers/bookmarks_webmcp_controller.ts` - Stimulus controller with WebMCP tools
- **API used**: `navigator.modelContext.registerTool()` in Stimulus

#### Angular + TypeScript
- **[angular/README.md](./angular/README.md)** - Note manager using `@mcp-b/global` with Angular services
- **Location**: `/angular`
- **Key file**: `angular/src/app/services/webmcp.service.ts` - Angular service with tool registration
- **API used**: `navigator.modelContext.registerTool()` via service
#### Phoenix LiveView (Elixir)
- **[phoenix-liveview/README.md](./phoenix-liveview/README.md)** - Counter + items with server-side state
- **Location**: `/phoenix-liveview`
- **Key files**:
  - `lib/webmcp_demo_web/live/counter_live.ex` - LiveView with state management
  - `assets/js/app.js` - WebMCP hook registration
- **API used**: `navigator.modelContext.registerTool()` via LiveView hooks

### Legacy Examples (Deprecated - DO NOT USE)
- **[relegated/README.md](./relegated/README.md)** - Old examples using deprecated MCP SDK
- **Warning**: These use the legacy `@modelcontextprotocol/sdk` API
- **Status**: Kept for reference only, not recommended for new projects

### Documentation
- **[CONTRIBUTING.md](./CONTRIBUTING.md)** - Contribution guidelines
- **[CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)** - Community standards
- **[CHANGELOG.md](./CHANGELOG.md)** - Version history and changes

## Common Development Tasks

### Running an Example

```bash
# Navigate to the example
cd vanilla  # or react, rails, angular
cd vanilla  # or react, rails, phoenix-liveview

# Install dependencies
pnpm install  # For JS examples
# OR
mix setup     # For Phoenix example

# Start development server
pnpm dev      # For JS examples
# OR
mix phx.server  # For Phoenix example
```

### Adding a New Example

1. **Check existing examples** first to avoid duplication
2. **Choose the right location**:
   - `/vanilla` for pure TypeScript/JavaScript
   - `/react` for React-based examples
   - `/rails` for Rails with Stimulus examples
   - `/angular` for Angular-based examples
3. **Create self-contained directory** with:
   - `README.md` - Documentation
   - `package.json` - Dependencies
   - `src/` - Source code
   - `vite.config.ts` - Build config
4. **Use modern WebMCP API**:
   - Vanilla: `@mcp-b/global` package
   - React: `@mcp-b/react-webmcp` package
5. **Follow patterns** in existing examples
6. **Document thoroughly** - explain what WebMCP concepts are demonstrated

See [CONTRIBUTING.md](./CONTRIBUTING.md) for detailed guidelines.

### Code Quality

```bash
# Type-check
pnpm typecheck

# Lint
pnpm lint

# Build
pnpm build
```

## Key Patterns

### Modern WebMCP Tool Registration

**Vanilla TypeScript:**
```typescript
import '@mcp-b/global';

navigator.modelContext.registerTool({
  name: 'my_tool',
  description: 'What this tool does',
  inputSchema: {
    type: 'object',
    properties: {
      param: { type: 'string', description: 'Description' }
    }
  },
  async execute(args) {
    return {
      content: [{ type: 'text', text: 'Result' }]
    };
  }
});
```

**React with Hooks:**
```tsx
import { useWebMCP } from '@mcp-b/react-webmcp';
import { z } from 'zod';

useWebMCP({
  name: 'my_tool',
  description: 'What this tool does',
  inputSchema: {
    param: z.string().describe('Description')
  },
  handler: async ({ param }) => {
    return { success: true };
  }
});
```

## File Locations

### Example Structure

Each example follows this structure:
```
example-name/
├── README.md          # What it demonstrates, how to run
├── package.json       # Dependencies (@mcp-b/* packages)
├── vite.config.ts     # Vite configuration
├── tsconfig.json      # TypeScript config
├── index.html         # HTML entry point
└── src/
    ├── main.ts(x)     # Application entry point
    └── ...            # Additional source files
```

### Key Files by Example

**Vanilla Example:**
- Entry: `vanilla/src/main.ts`
- Config: `vanilla/vite.config.ts`
- Types: `vanilla/src/types.ts`

**React Example:**
- Entry: `react/src/main.tsx`
- Root: `react/src/App.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WebMCP-org/examples](https://github.com/WebMCP-org/examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
