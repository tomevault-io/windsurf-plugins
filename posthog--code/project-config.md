---
trigger: always_on
description: - Monorepo with pnpm workspaces and turbo
---

# PostHog Code Development Guide

## Project Structure

- Monorepo with pnpm workspaces and turbo
- `apps/code` - PostHog Code Electron desktop app (React + Vite)
- `apps/cli` - CLI tool (thin wrapper around @posthog/core)
- `apps/mobile` - React Native mobile app (Expo)
- `packages/agent` - TypeScript agent framework wrapping Claude Agent SDK
- `packages/core` - Shared business logic for jj/GitHub operations
- `packages/electron-trpc` - Custom tRPC package for Electron IPC
- `packages/shared` - Shared utilities (Saga pattern, etc.) used across packages

## Commands

- `pnpm install` - Install all dependencies
- `pnpm dev` - Run both agent (watch) and code app via phrocs
- `pnpm dev:mprocs` - Run both agent (watch) and code app via mprocs
- `pnpm dev:agent` - Run agent package in watch mode only
- `pnpm dev:code` - Run code desktop app only
- `pnpm build` - Build all packages (turbo)
- `pnpm typecheck` - Type check all packages
- `pnpm lint` - Lint and auto-fix with biome
- `pnpm format` - Format with biome
- `pnpm test` - Run tests across all packages

### Code App Specific

- `pnpm --filter code test` - Run vitest tests
- `pnpm --filter code typecheck` - Type check code app
- `pnpm --filter code package` - Package electron app
- `pnpm --filter code make` - Make distributable

### Agent Package Specific

- `pnpm --filter agent build` - Build agent with tsup
- `pnpm --filter agent dev` - Watch mode build
- `pnpm --filter agent typecheck` - Type check agent

### Shared Package Specific

- `pnpm --filter @posthog/shared build` - Build shared with tsup
- `pnpm --filter @posthog/shared dev` - Watch mode build
- `pnpm --filter @posthog/shared typecheck` - Type check shared

## Code Style

- Prefer writing our own solution over adding external packages when the fix is simple
- Keep functions focused with single responsibility
- Biome for linting and formatting (not ESLint/Prettier)
- 2-space indentation, double quotes
- No `console.*` in source - use logger instead (logger files exempt)
- Path aliases required in renderer code - no relative imports
  - `@features/*`, `@components/*`, `@stores/*`, `@hooks/*`, `@utils/*`, `@renderer/*`, `@shared/*`, `@api/*`
- Main process path aliases: `@main/*`, `@api/*`, `@shared/*`
- TypeScript strict mode enabled
- Tailwind CSS classes should be sorted (biome `useSortedClasses` rule)

### Services Over Hooks for Business Logic

Put data-fetching logic and derivation in main process services, not renderer hooks. Hooks should be thin wrappers around a single tRPC query. If a hook orchestrates multiple queries and derives a result, that logic belongs in a service exposed via tRPC so it can be reused from both the main process and the renderer.

### Small Focused Components

Extract distinct UI concerns into their own components instead of building long inline ternary chains or conditional blocks. If a section of JSX handles its own logic (e.g. icon selection based on state), pull it into a named component next to where it's used. Keep render functions short and scannable.

### Async Cleanup Ordering

When tearing down async operations that use an AbortController, always abort the controller **before** awaiting any cleanup that depends on it. Otherwise you get a deadlock: the cleanup waits for the operation to stop, but the operation won't stop until the abort signal fires.

```typescript
// WRONG - deadlocks if interrupt() waits for the operation to finish
await this.interrupt();          // hangs: waits for query to stop
this.abortController.abort();    // never reached

// RIGHT - abort first so the operation can actually stop
this.abortController.abort();    // cancels in-flight HTTP requests
await this.interrupt();          // resolves because the query was aborted
```

### Avoid Barrel Files

- Do not make use of index.ts

Barrel files:

- Break tree-shaking
- Create circular dependency risks
- Hide the true source of imports
- Make refactoring harder

Import directly from source files instead.

## Architecture

See [ARCHITECTURE.md](./apps/code/ARCHITECTURE.md) for detailed patterns (DI, services, tRPC, state management).

### Electron App (apps/code)

- **Main process** (`src/main/`) - Services own all business logic, orchestration, polling, data fetching, and system I/O
- **Renderer process** (`src/renderer/`) - React app with Zustand stores holding pure UI state and thin action wrappers over tRPC
- **IPC**: tRPC over Electron IPC (type-safe via @posthog/electron-trpc)
- **DI**: InversifyJS in both processes (`src/main/di/`, `src/renderer/di/`)
- **Testing**: Vitest with React Testing Library

### Agent Package (packages/agent)

- Wraps `@anthropic-ai/claude-agent-sdk`
- Git worktree management in `worktree-manager.ts`
- PostHog API integration in `posthog-api.ts`
- Task execution and session management

### CLI Package (packages/cli)

- **Dumb shell, imperative core**: CLI commands should be thin wrappers that call `@posthog/core`
- All business logic belongs in `@posthog/core`, not in CLI command files
- CLI only handles: argument parsing, calling core, formatting output
- No data transformation, tree building, or complex logic in CLI

### Core Package (packages/core)

- Shared business logic for jj/GitHub operations

### Shared Package (packages/shared)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PostHog/code](https://github.com/PostHog/code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
