---
trigger: always_on
description: Tasks Map is an Obsidian plugin that visualizes tasks as an interactive graph.
---

# AGENTS.md

## Project Overview

Tasks Map is an Obsidian plugin that visualizes tasks as an interactive graph.
Built with TypeScript, React 19, ReactFlow, and i18next. Bundled with esbuild
to CommonJS. The Obsidian API is external (not bundled).

## Commands

| Command                 | Purpose                                                |
| ----------------------- | ------------------------------------------------------ |
| `npm run dev`           | Watch mode development build                           |
| `npm run build`         | Typecheck (`tsc --noEmit`) + production esbuild bundle |
| `npm test`              | Run all Jest tests                                     |
| `npm run test:watch`    | Run tests in watch mode                                |
| `npm run test:coverage` | Run tests with coverage report                         |
| `npm run lint`          | ESLint check on `src/`                                 |
| `npm run lint:fix`      | ESLint autofix                                         |
| `npm run format`        | Prettier check                                         |
| `npm run format:fix`    | Prettier autofix                                       |

### Running a single test

```sh
npx jest test/task-factory.test.ts          # single file
npx jest -t "extracts emoji-format ID"      # by test name pattern
npx jest test/filter-tasks.test.ts -t "tag" # file + name pattern
```

## Project Structure

```
src/
├── main.tsx                 # Plugin entry point (extends Obsidian Plugin)
├── components/              # React UI components (task-node, star-button, etc.)
├── contexts/                # React contexts (AppContext, TagsContext)
├── hooks/                   # Custom hooks (useApp, useSummaryRenderer)
├── i18n/                    # i18next setup + locales (en, nl, zh-CN)
├── lib/                     # Pure business logic (task-factory, filter, utils)
├── settings/                # Obsidian settings tab
├── types/                   # Type definitions (BaseTask, DataviewTask, NoteTask)
└── views/                   # Obsidian ItemView wrappers for the React graph
test/
├── *.test.ts                # Unit tests (mirror src/lib/ and src/types/ names)
├── mocks/                   # Manual mocks for obsidian, react, reactflow, etc.
└── fixture/                 # Sample Obsidian vault for integration tests
```

## Code Style

### Formatting (Prettier)

- Double quotes everywhere (imports, strings, JSX).
- Semicolons always.
- Trailing commas in ES5 positions.
- 2-space indentation, 80-character line width.
- Arrow parens always: `(x) => ...`.
- LF line endings.

### Imports

Three-tier grouping (no blank lines between groups):

1. External libraries (`react`, `reactflow`, `obsidian`, `lucide-react`)
2. Internal absolute paths (`src/hooks/hooks`, `src/types/task`, `src/lib/utils`)
3. Relative paths (`./task-details`, `../i18n`)

Named imports are preferred. Default imports only for React, ReactFlow, and
primary component exports. Multi-symbol imports use destructured form with
trailing commas.

```typescript
import React, { useState, useCallback } from "react";
import { Handle, Position } from "reactflow";
import { useApp } from "src/hooks/hooks";
import { BaseTask } from "src/types/task";
import { TaskDetails } from "./task-details";
```

### Types

- `interface` for object shapes and component props.
- `type` for unions, aliases, and generic wrappers.
- Props interfaces: `ComponentNameProps` (e.g., `StarButtonProps`).
- Explicit return types only when non-obvious (abstract methods, hooks with
  specific return types, `void` returns in settings). Otherwise rely on
  inference.
- Prefix unused parameters with underscore: `_app`, `_newStatus`.

```typescript
interface TaskNodeData { ... }               // object shape
type TaskStatus = "todo" | "in_progress" | "canceled" | "done";  // union
type TaskNode = Node<TaskNodeData, "task">;   // generic alias
```

### Naming Conventions

| Kind                        | Convention              | Examples                                       |
| --------------------------- | ----------------------- | ---------------------------------------------- |
| Variables, functions, hooks | camelCase               | `filterState`, `useApp`, `handleDeleteTask`    |
| Classes                     | PascalCase              | `TaskFactory`, `BaseTask`, `NoteTask`          |
| React components            | PascalCase              | `TaskNode`, `StarButton`, `GuiOverlay`         |
| Interfaces, types           | PascalCase              | `FilterState`, `TasksMapSettings`              |
| Constants, regex patterns   | UPPER_CASE              | `VIEW_TYPE`, `DEFAULT_SETTINGS`, `TAG_PATTERN` |
| Files                       | kebab-case              | `task-node.tsx`, `filter-tasks.ts`             |
| View files (exception)      | PascalCase              | `TaskMapGraphView.tsx`                         |
| Test files                  | kebab-case + `.test.ts` | `task-factory.test.ts`                         |

### Exports

- At most one `default` export per file for the primary component or class.
- Named exports for everything else (types, constants, utilities, small
  components).

### React Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NicoKNL/tasks-map](https://github.com/NicoKNL/tasks-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
