---
trigger: always_on
description: Development conventions, code style, and best practices for the Kaneo project
---

# Development Conventions

This document outlines coding standards, conventions, and best practices for the Kaneo project.

## Code Style

### Biome Configuration

Kaneo uses **Biome** for linting and formatting. Configuration is in `biome.json`.

#### Formatting Rules

- **Indentation**: Tabs (not spaces) for TypeScript/TSX
- **JavaScript**: Spaces for indentation (legacy support)
- **Quote Style**: Double quotes (`"`)
- **Semicolons**: Required

```typescript
// Good: Tabs, double quotes, semicolons
function example() {
	const value = "hello";
	return value;
}

// Bad: Spaces, single quotes, no semicolons
function example() {
  const value = 'hello'
  return value
}
```

#### Linting Rules

Key Biome rules enabled:

- `noParameterAssign`: Don't reassign function parameters
- `useAsConstAssertion`: Use `as const` for literal types
- `useDefaultParameterLast`: Default parameters must be last
- `useSelfClosingElements`: Use self-closing JSX elements
- `useSingleVarDeclarator`: One variable per declaration
- `noInferrableTypes`: Don't add explicit types that can be inferred

```typescript
// Good: Follows Biome rules
const items = ["a", "b"] as const;
function greet(name: string, greeting = "Hello") {
	return `${greeting}, ${name}`;
}
const x = 1;
const y = 2;

// Bad: Violates Biome rules
const items: string[] = ["a", "b"];
function greet(greeting = "Hello", name: string) {
	return `${greeting}, ${name}`;
}
const x = 1, y = 2;
```

### Running Linter

```bash
# Check and auto-fix
pnpm lint

# Check only (no fixes)
pnpm biome check .
```

## TypeScript Conventions

### Type Definitions

- **Types**: Prefer types for all type definitions (object shapes, unions, intersections, computed types)
- **Interfaces**: Only use interfaces when extending/merging is needed (rare)
- **Inference**: Prefer type inference when types are obvious

```typescript
// Good: Type for object shape
type Task = {
	id: string;
	title: string;
	status: string;
};

// Good: Type for union
type Status = "to-do" | "in-progress" | "done";

// Good: Type inference
const tasks: Task[] = []; // Explicit for arrays
const count = tasks.length; // Inferred
```

### File Naming

- **Components**: PascalCase: `TaskCard.tsx`
- **Utilities**: kebab-case: `format-date.ts`
- **Hooks**: camelCase with `use` prefix: `use-task.ts`
- **Types**: kebab-case: `task-types.ts`

```
components/
├── TaskCard.tsx          # Component
├── task-card.tsx         # Also acceptable
└── utils/
    └── format-date.ts   # Utility function
```

## Git Conventions

### Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <description>

[optional body]

[optional footer]
```

#### Commit Types

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `refactor:` - Code refactoring (no feature/fix)
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks (deps, config, etc.)
- `style:` - Code style changes (formatting, etc.)

#### Examples

```bash
feat: add bulk task operations
fix: resolve calendar date selection bug
docs: update deployment guide
refactor: simplify task controller logic
chore: update dependencies
```

### Branch Naming

Use descriptive branch names with prefixes:

- `feat/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation
- `refactor/` - Refactoring
- `chore/` - Maintenance

```bash
feat/bulk-task-operations
fix/calendar-date-bug
docs/update-deployment-guide
```

## Import Organization

Biome automatically organizes imports. Follow these patterns:

### Import Order

1. External packages
2. Internal packages (`@/` aliases)
3. Relative imports

```typescript
// Good: Organized imports
import { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { TaskCard } from "./task-card";
```

### Import Style

- Use named imports when possible
- Group related imports
- Remove unused imports (Biome does this automatically)

```typescript
// Good: Named imports
import { useState, useEffect } from "react";
import { Button, Input } from "@/components/ui";

// Avoid: Default imports when named available
import React from "react"; // Prefer named imports
```

## File Structure

### Component Files

```typescript
// 1. Imports (external, then internal)
import { useState } from "react";
import { Button } from "@/components/ui/button";

// 2. Types
type ComponentProps = {
	title: string;
};

// 3. Component
export function Component({ title }: ComponentProps) {
	// Implementation
}

// 4. Exports (if needed)
export default Component;
```

### API Controller Files

```typescript
// 1. Imports
import db from "../../database";
import { taskTable } from "../../database/schema";

// 2. Function
export default async function getTask(id: string) {
	// Implementation
}
```

## Error Handling

### Backend

Use Hono's HTTPException:

```typescript
import { HTTPException } from "hono/http-exception";

if (!task) {
	throw new HTTPException(404, { message: "Task not found" });
}
```

### Frontend

Use try-catch with user-friendly messages:

```typescript
import { toast } from "sonner";

try {
	await updateTask(taskId, data);
	toast.success("Task updated");
} catch (error) {
	toast.error(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/usekaneo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
