---
trigger: always_on
description: This file is injected into every agent session for this project. Keep it short.
---

# Palot Agent Instructions

## Purpose of This File

This file is injected into every agent session for this project. Keep it short.
Only add entries here if an agent is likely to get stuck or repeat a mistake without them.
Do NOT add one-time setup notes, general knowledge, or things discoverable from config files.

## Project Structure

- **Monorepo**: Turborepo + Bun workspaces (Bun 1.3.8)
- **`packages/ui`**: Shared shadcn/ui component library (`@palot/ui`)
- **`packages/configconv`**: Universal agent config converter library (`@palot/configconv`) -- converts between Claude Code, OpenCode, and Cursor formats
- **`packages/configconv-cli`**: Thin CLI wrapper (`configconv`) for the converter library
- **`apps/desktop`**: Electron 40 + Vite + React 19 desktop app (via `electron-vite`)
- **`apps/server`**: Bun + Hono backend -- used only in browser-mode dev (`dev:web`), NOT bundled with Electron

### Desktop App Layout (`apps/desktop/src/`)

- **`main/`** -- Electron main process (Node.js): window management, IPC handlers, OpenCode server lifecycle, filesystem reads
- **`preload/`** -- Electron preload bridge: exposes `window.palot` API via `contextBridge`
- **`renderer/`** -- React app (browser context): components, hooks, services, atoms (Jotai)

## Skills

Project-specific skills live in `.agents/skills/`. Load a skill before starting
work that matches its domain -- they contain patterns and footguns that override
generic knowledge.

| Skill | When to load |
|---|---|
| `react-best-practices` | Writing or reviewing renderer components, optimizing re-renders or bundle size |

## Commands

- **Electron dev**: `cd apps/desktop && bun run dev` (electron-vite, renderer on port 1420)
- **Browser-only dev**: `cd apps/desktop && bun run dev:web` (Vite only, needs `apps/server` running)
- **Backend server** (browser mode only): `cd apps/server && bun run dev` (port 3100)
- **Lint check**: `bun run lint` (from root)
- **Lint/format fix**: `bun run lint:fix` or `bunx biome check --write .` (from root)
- **Type check all**: `bun run check-types` (from root, via Turborepo)
- **Type check desktop**: `cd apps/desktop && bun run check-types` (uses `tsgo`)
- **Run all tests**: `cd packages/configconv && bun test`
- **Run single test file**: `cd packages/configconv && bun test test/converter/config.test.ts`
- **Run tests by name**: `cd packages/configconv && bun test --grep "converts model"`
- **Rebuild server types**: `cd apps/server && bun run build:types` (required after adding server routes)
- **Add UI component**: `cd packages/ui && bunx shadcn@latest add <component>`
- **Package**: `cd apps/desktop && bun run package` (or `package:linux`, `package:mac`, `package:win`, `package:all`)
- **Package without code signing (macOS)**: `CSC_IDENTITY_AUTO_DISCOVERY=false cd apps/desktop && bun run package:mac`
- **Changeset -- add**: `bun changeset` (interactive -- pick packages, bump type, write description)
- **Changeset -- version**: `bun run version-packages` (applies pending changesets, bumps versions, updates changelogs)

## Code Style

### Formatting (enforced by Biome 2.3.14)

- Tabs for indentation (width 2), line width 100, LF line endings
- Double quotes, no semicolons, trailing commas everywhere
- Arrow functions always use parentheses: `(x) => x`
- Run `bunx biome check --write .` from root to auto-fix

### Imports

- `node:` protocol for all Node.js builtins: `import path from "node:path"`
- Use `import type { ... }` for type-only imports (Biome warns otherwise)
- Order: external packages first, then internal/relative imports (no blank line between)
- Main process: `node:` builtins first, then `electron`, then local
- Renderer: `@palot/ui` -> `@tanstack/*` -> `lucide-react` -> `react` -> local atoms/hooks/services

### Naming Conventions

- **Files**: `kebab-case.ts` / `kebab-case.tsx` everywhere
- **Functions/variables**: `camelCase` -- `createLogger()`, `fetchDiscovery()`
- **Components**: `PascalCase` -- `ChatView`, `AppSidebar`, `CommandPalette`
- **Types/interfaces**: `PascalCase` -- `DiscoveredProject`, `AgentStatus`
- **Props**: `ComponentNameProps` -- `ChatViewProps`, `AppSidebarProps`
- **Module-level constants**: `UPPER_SNAKE_CASE` -- `FRAME_BUDGET_MS`, `OPENCODE_PORT`
- **Jotai atoms**: `camelCaseAtom` -- `sessionIdsAtom`, `serverUrlAtom`
- **Atom families**: `camelCaseFamily` -- `sessionFamily`, `partsFamily`

### Types

- Prefer `interface` for object shapes, `type` for unions/aliases
- Export types only when used across modules
- Props: named interface for complex props, inline destructured type for small sub-components
- UI library uses `React.ComponentProps<"element">` intersection pattern for wrapper components

### React Patterns

- Functional components only, no class components
- State: **Jotai atoms** (NOT Zustand -- codebase has migrated). Store in `renderer/atoms/`
- Thin hook wrappers around atoms (e.g., `useAgents()` returns `useAtomValue(agentsAtom)`)
- Use `memo()` with named function expressions for perf-critical sub-components
- Custom hooks return objects, not arrays
- Named exports everywhere -- no default exports (except Hono route modules and Bun server entry)

### Error Handling

- No custom error classes -- use `new Error("descriptive message")`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ItsWendell/palot](https://github.com/ItsWendell/palot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
