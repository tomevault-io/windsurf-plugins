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
- **Client**: React 19 + Vite 8 + TanStack Router + Zustand + shadcn/ui + Tailwind CSS v4, with PWA support (packages/client)
- **SDK**: Shared types, protocols, transport layer, WebSocket namespaces, and REST clients (packages/sdk)
- **Client Electron**: Electron desktop wrapper around the client (packages/client-electron)
- **Browser Extension**: Chrome extension for browser automation (packages/browser)
- **External Tools**: TypeScript tool modules, separately versioned and distributed (tools/). Each tool is a directory with `tool.ts`, `package.json`, and `VERSION`.
- **Sandbox CLI**: Interactive CLI for intercepting and simulating LLM responses in a running server, enabling end-to-end testing without real API calls (packages/sandbox-cli).

## Build Commands

```bash
# Install dependencies
bun install

# Development (runs both server and client)
bun run dev

# Development with HTTPS
bun run dev:https

# Development - server only
bun run dev:server
# Alias
bun run dev:be

# Development - client only
bun run dev:client
bun run dev:client:https

# Development - Electron desktop
bun run dev:electron

# Build all packages
bun run build

# Build tools
bun run build:tools

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

# Preview production client build
bun run preview
bun run preview:https

# Build Electron desktop app
bun run electron:build
bun run electron:build:mac:local
bun run electron:build:mac:release
bun run electron:build:win

# Start sandbox CLI
bun run sandbox
```

## Lint Commands

```bash
# Run ESLint
bun run lint

# Run ESLint with auto-fix
bun run lint:fix
```

ESLint uses flat config (`eslint.config.js`) with `typescript-eslint`, `eslint-plugin-react`, and `eslint-plugin-react-hooks`. The `tools/` directory is linted with its own Bun globals config.

## Test Commands

```bash
# Run all tests (server + sdk + tools + client)
bun run test

# Server tests (Bun test runner)
bun run test:server
bun run test:server:coverage

# Client tests (Vitest)
bun run test:client

# Tool tests (Bun test runner)
bun run test:tools
```

- **Server**: Uses `bun:test` with `describe`/`test`/`expect`/`beforeEach`/`afterEach`. Test helpers in `packages/server/tests/helpers/` with import aliases (`#tests/db`, `#tests/factories`, `#tests/seed`, `#tests/mocks`, `#tests/test-dir`).
- **Client**: Uses Vitest with `describe`/`test`/`expect`/`beforeEach`. Zustand stores tested via `useStore.getState()` directly.
- **Tools**: Uses `bun:test` with shared `test-utils.ts` providing `createMockContext`, `VirtualFS`, and `WORKSPACE` for virtual filesystem testing.

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
- **Constants**: SCREAMING_SNAKE_CASE for env-derived (`JEAN2_LLM_MAX_TOKENS`), camelCase otherwise
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
- Server data via TanStack Query hooks (`packages/client/src/hooks/queries/`)
- Routing via TanStack Router with file-based code splitting
- UI components built on shadcn/ui (Radix primitives + Tailwind)

```typescript
interface Props {
  session: Session;
  onSendMessage: (content: string) => void;
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jean2ai/jean2](https://github.com/jean2ai/jean2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
