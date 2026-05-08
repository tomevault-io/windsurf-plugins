---
trigger: always_on
description: Guidelines for AI coding agents working in this repository.
---

# AGENTS.md

Guidelines for AI coding agents working in this repository.

## Project Overview

Jean2 is an AI Agent monorepo built with TypeScript, Bun, React, and Hono.

- **Runtime**: Bun
- **Monorepo**: Workspace-based with packages in `packages/`
- **Server**: Hono + AI SDK with multi-provider support (packages/server)
- **Client**: React 19 + Vite + TanStack Router + Zustand + shadcn/ui + Tailwind CSS v4 (packages/client)
- **SDK**: Shared types, protocols, transport layer, and REST clients (packages/sdk)
- **Client Electron**: Electron desktop wrapper around the client (packages/client-electron)
- **Client Tauri**: Tauri (Rust) native app for mobile and desktop (packages/client-tauri)
- **External Tools**: Independent executable tool scripts, separately versioned and distributed (tools/)

## Build Commands

```bash
# Install dependencies
bun install

# Development (runs both server and client)
bun run dev

# Development - server only
bun run dev:server
# Alias
bun run dev:be

# Development - client only
bun run dev:client

# Development - Electron desktop
bun run dev:electron

# Build all packages
bun run build

# Type check all packages
bun run typecheck

# Build server binary (current platform)
bun run build:bin

# Build server binary for specific platform
bun run build:bin:macos
bun run build:bin:linux
bun run build:bin:windows

# Build server package + binary
bun run build:all

# Build Electron desktop app
bun run electron:build
bun run electron:build:mac:local
bun run electron:build:mac:release
bun run electron:build:win

# Build Tauri native app (from client package)
bun run tauri:build:windows
```

## Lint Commands

```bash
# Run ESLint
bun run lint

# Run ESLint with auto-fix
bun run lint:fix
```

ESLint uses flat config (`eslint.config.js`) with `typescript-eslint`, `eslint-plugin-react`, and `eslint-plugin-react-hooks`. The `tools/` directory is excluded from linting.

## Test Commands

No test framework is currently configured. Tests would follow the pattern:
```bash
# Run all tests (when configured)
bun test

# Run a single test file
bun test path/to/test.file.ts
```

## Code Style

### Imports

- Use `import type` for type-only imports
- Group imports: external libraries first, then internal packages (`@jean2/*`), then local (`@/`)
- Use `@/*` path alias for relative imports within the same package

```typescript
import { useState, useEffect } from 'react';
import type { Session, Message } from '@jean2/sdk';
import { fetchMessages } from '@/store';
import './styles.css';
```

### Naming Conventions

- **Variables/Functions**: camelCase (`getUserById`, `isLoading`)
- **Components**: PascalCase (`ChatView`, `SessionList`)
- **Types/Interfaces**: PascalCase (`Session`, `ToolDefinition`)
- **Type aliases**: PascalCase (`SessionStatus`, `ToolRuntime`)
- **Constants**: SCREAMING_SNAKE_CASE for env-derived (`LLM_MAX_TOKENS`), camelCase otherwise
- **Files**: camelCase for modules (`agent.ts`), PascalCase for components (`ChatView.tsx`)

### TypeScript

- Strict mode enabled
- Prefer `interface` for object shapes, `type` for unions/primitives
- Use explicit return types for exported functions
- Avoid `any`; use `unknown` when type is uncertain
- Use `as const` for literal objects that should be immutable
- Unused vars prefixed with `_` (e.g., `_e`, `_sessionId`)

```typescript
export interface ChatOptions {
  sessionId: string;
  messages: Message[];
}

export type SessionStatus = 'active' | 'closed';

export async function getTool(name: string): Promise<DiscoveredTool | null> {
  // ...
}
```

### React

- React 19 with React Compiler (configured in Vite via `babel-plugin-react-compiler`)
- Functional components with hooks
- Destructure props in function signature
- Use `export default` for page/container components
- Named exports for utility components/hooks
- State management via Zustand stores (`packages/client/src/stores/`)
- Routing via TanStack Router with file-based code splitting
- UI components built on shadcn/ui (Radix primitives + Tailwind)

```typescript
interface Props {
  session: Session;
  onSendMessage: (content: string) => void;
}

export default function ChatView({ session, onSendMessage }: Props) {
  const [input, setInput] = useState('');
  // ...
}
```

### Error Handling

- Return error objects with `success` boolean for tool execution
- Use try/catch for async operations; type catch as `unknown`
- Log errors with context before returning

```typescript
export interface ToolResult {
  success: boolean;
  result?: unknown;
  error?: string;
}

try {
  const result = await executeOperation();
  return { success: true, result };
} catch (err: unknown) {
  const message = err instanceof Error ? err.message : String(err);
  console.error('Operation failed:', message);
  return { success: false, error: message };
}
```

### Formatting

- No comments unless absolutely necessary for complex logic
- 2-space indentation
- Single quotes for strings (double quotes only when required)
- Trailing commas in multiline structures

### Environment Variables

- Prefix with `JEAN2_` for application settings
- Access via `process.env.VAR_NAME`
- Provide defaults with `||` or `??`

```typescript
const JEAN2_LLM_MAX_TOKENS = parseInt(process.env.JEAN2_LLM_MAX_TOKENS || '4096', 10);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rabbyte-tech/jean2](https://github.com/rabbyte-tech/jean2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
