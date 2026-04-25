---
trigger: always_on
description: handleSubmit();
---

# Nomendex
Nomendex is a desktop application for working with notes, tasks, and AI agents.

It is NOT an Electron app. It is a desktop application built with Bun and React. Bun is used as a sidebar within a MacOS Swift container.

## Build Commands

- `bun run dev` - Start development server with hot reload (port 1234 default)
- `bun run build` - Run full build validation (Tailwind CSS, ESLint, TypeScript checking)

**CRITICAL**: Always run `bun run build` after making changes to validate TypeScript and linting.

## Architecture Overview

### Features
The application has two built-in features:
- **Todos** (`/src/features/todos/`) - Task management with kanban, projects, archive
- **Notes** (`/src/features/notes/`) - Markdown note-taking with file browser

### Core Technologies
- **Runtime**: Bun (not Node.js) - use `bun` for all commands
- **Framework**: React 19 with server-side rendering via Bun.serve()
- **Routing**: React Router (react-router-dom)
- **Styling**: Tailwind CSS + shadcn/ui components + Theme system
- **Type Safety**: TypeScript with Zod for runtime validation
- **State Management**: React hooks with feature-specific API hooks

### Workspace Data Storage
Each user can create a workspace which is a folder on the filesystem that contains all user data for that workspace.

**Global Config** (`src/storage/global-config.ts`):
- Stored at `~/Library/Application Support/com.firstloop.nomendex/config.json`
- Tracks all workspaces and which one is active
```typescript
interface GlobalConfig {
    workspaces: WorkspaceInfo[];  // id, path, name, createdAt, lastAccessedAt
    activeWorkspaceId: string | null;
}
```

**Workspace Paths** (`src/storage/root-path.ts`):
- `getRootPath()`, `getTodosPath()`, `getNotesPath()`, `getAgentsPath()`, `getSkillsPath()`, `getUploadsPath()`
- `hasActiveWorkspace()` - check before accessing paths

**Workspace Directory Structure**:
```
/path/to/workspace/
├── todos/              # Todo items (markdown + YAML frontmatter)
├── notes/              # Notes
├── uploads/            # Images and attachments
├── agents/             # Agent configurations
├── .claude/skills/     # Custom skills
└── workspace.json      # UI state (tabs, theme, etc.)
```

**Workspace State** (`src/types/Workspace.ts`):
- Stored in `{workspace}/workspace.json`
- Contains tabs, activeTabId, sidebarOpen, themeName, mcpServerConfigs

**Key Hooks**:
- `useWorkspace.tsx` - Manages workspace UI state (tabs, theme)
- `useWorkspaceSwitcher.ts` - Switch/add/remove workspaces

**API Routes**:
- `/api/workspaces` - List all workspaces
- `/api/workspaces/active` - Get active workspace
- `/api/workspaces/switch` - Switch workspace (triggers reload)
- `/api/workspace` - GET/POST workspace state

## Feature Structure

Each feature follows this pattern:

```
src/features/[feature]/
├── index.ts           # Feature definition, types, and exports
├── fx.ts              # Server-side function implementations
├── view.tsx           # Main view component
├── browser-view.tsx   # Browser/list view component
└── commands.tsx       # Command palette commands
```

### API Pattern

#### Server-side routes (`/src/server-routes/`)
```typescript
// todos-routes.ts
import { getTodos, createTodo } from "@/features/todos/fx";

export const todosRoutes = {
    "/api/todos/list": {
        async POST(req: Request) {
            const args = await req.json();
            return Response.json(await getTodos(args));
        },
    },
};
```

#### Client-side API hooks (`/src/hooks/`)
```typescript
// useTodosAPI.ts
export const todosAPI = {
    getTodos: (args) => fetchAPI<Todo[]>("list", args),
    createTodo: (args) => fetchAPI<Todo>("create", args),
};

// Hook wrapper for React components
export function useTodosAPI() {
    return todosAPI;
}
```

#### Using in components
```typescript
// In React components (use hook)
const api = useTodosAPI();
const todos = await api.getTodos({});

// Outside React (use standalone API)
import { todosAPI } from "@/hooks/useTodosAPI";
const todos = await todosAPI.getTodos({});
```

## Routing

Uses React Router with routes defined in `App.tsx`:
- `/` - WorkspacePage (main workspace with tabs)
- `/settings` - SettingsPage
- `/sync` - SyncPage
- `/mcp-clients` - McpClientsPage
- `/tests` - TestsPage

## Design System

### Theme System
```typescript
import { useTheme } from "@/hooks/useTheme";
const { currentTheme } = useTheme();
```

### Component Library
- Use shadcn/ui components from `@/components/ui/*`
- Focus on composability and reusability
- Keep animations subtle (scale-[1.02], not scale-110)

### Keyboard Shortcuts
```tsx
import { KeyboardIndicator } from "@/components/KeyboardIndicator";
<Button><KeyboardIndicator keys={['cmd', 'n']} /> New</Button>
```

### Application Design Principles
- Reuse previously defined components over creating net new code. Especially when reusing dialogs and similar.
- Try to hide buttons to keep visual complexity down. Reveal on hover is a good technique.
- Always use `useTheme` for styling
- Add commands for functionality over discrete buttons

## Critical Implementation Rules

### Type Safety
- **NEVER use `any` type** - build will fail
- Use Zod schemas and infer types from them
- Validate all inputs and outputs at runtime

### Centralized Types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firstloophq/nomendex](https://github.com/firstloophq/nomendex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
