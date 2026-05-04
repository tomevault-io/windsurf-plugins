---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Documentation Map

### CLAUDE.md Files

- **Root** (this file) - Monorepo essentials + critical rules
- `apps/app/CLAUDE.md` - Full-stack app integration
- `apps/app/src/client/CLAUDE.md` - Frontend patterns
- `packages/agent-cli-sdk/CLAUDE.md` - AI CLI SDK
- `packages/agentcmd-workflows/CLAUDE.md` - Workflow SDK

### .agent/docs/ Reference Guides

- `backend-patterns.md` - Domain services, routes, error handling
- `frontend-patterns.md` - React patterns, state management
- `workflow-system.md` - Workflow engine, Inngest integration
- `websocket-architecture.md` - WebSocket patterns, EventBus
- `database-guide.md` - Prisma patterns, migrations
- `troubleshooting.md` - Common issues and solutions
- `deployment.md` - Build, deploy, production
- `architecture-decisions.md` - Why we chose these technologies
- `testing-best-practices.md` - Testing patterns
- `branding.md` - Project identity, design system

## Project Overview

**Turborepo monorepo** for AI agent workflow tools:

- **`apps/app`**: Full-stack application (React + Vite frontend, Fastify backend)
- **`apps/appsite`**: Marketing site (Next.js)
- **`packages/agent-cli-sdk`**: TypeScript SDK for AI CLI orchestration
- **`packages/agentcmd-workflows`**: Workflow utilities library

**Tech Stack:**

- Frontend: React 19, Vite, TanStack Query, Zustand, Tailwind v4
- Backend: Fastify, Prisma (SQLite), Inngest, WebSocket
- Build: Turborepo, pnpm, TypeScript 5.9

## Critical Rules

### Import Conventions

✅ **DO** - No file extensions, @/ aliases only:

```typescript
import { foo } from "./bar";
import { getProjectById } from "@/server/domain/project/services/getProjectById";
```

❌ **DON'T** - File extensions, relative paths:

```typescript
import { foo } from "./bar.js";
import { getProjectById } from "../../../domain/project/services/getProjectById";
```

✅ **DO** - Top-level imports:

```typescript
import type { PhaseDefinition } from "agentcmd-workflows";
```

❌ **DON'T** - Inline imports:

```typescript
function foo(phase: import("agentcmd-workflows").PhaseDefinition) {}
```

### React Best Practices

✅ **DO** - Primitives only in useEffect deps:

```typescript
const { userId, projectId } = project;
useEffect(() => {
  fetchData(userId, projectId);
}, [userId, projectId]);
```

❌ **DON'T** - Objects cause infinite loops:

```typescript
useEffect(() => {
  fetchData(project);
}, [project]);
```

✅ **DO** - Immutable Zustand updates:

```typescript
set((state) => ({
  messages: [...state.messages, newMessage],
}));
```

❌ **DON'T** - Mutations:

```typescript
set((state) => {
  state.messages.push(newMessage);
  return { messages: state.messages };
});
```

### Code Organization

**One export per file:**

- **Services** - server-side only (`domain/*/services/*.ts`)
- **Utils** - everywhere (client and server utils)
- File name matches export: `createProject.ts` → `export function createProject()`
- **Shared types** - import from `.types.ts` files
- **Private types** - keep in file (not exported)
- **Non-shared exported types** - can stay in file

**Utils Organization:**

Scan before creating (one export per file):
- `shared/utils/` - Pure functions used by BOTH frontend and backend
- `server/utils/` - Server-only utilities (cross-domain)
- `server/domain/*/utils/` - Domain-specific server utilities
- `client/utils/` - Client-only utilities (browser APIs, DOM, UI)

```
domain/
├── project/services/
│   ├── getProjectById.ts       # exports getProjectById only
│   ├── createProject.ts        # exports createProject + non-shared types
│   └── types.ts                # shared types used by multiple services
```

**CRUD Gold Standard:** Follow Prisma naming patterns for domain services:

- `get{Entity}` (findUnique), `get{Entity}By` (findFirst), `get{Entity}s` (findMany)
- `create{Entity}`, `update{Entity}`, `upsert{Entity}`, `delete{Entity}`
- See `.agent/docs/backend-patterns.md` for comprehensive patterns
- Reference: `apps/app/src/server/domain/workflow/services/definitions/`

**File structure:**

- Imports → Module constants/private types → `// PUBLIC API` separator → Public exports with JSDoc → `// PRIVATE HELPERS` separator → Private functions
- One function export per file, file name matches export
- Shared types from `.types.ts`, private types stay in file
- Route files and `*.types.ts` files are exceptions

**See:** `.agent/docs/backend-patterns.md` for detailed examples and patterns

### Schema Organization

**Hybrid approach** - share validation schemas, keep model interfaces separate:

```typescript
// ✅ Share enums and validation
// apps/app/src/shared/schemas/workflow.schemas.ts
export const workflowStatusSchema = z.enum(["pending", "running", "completed"]);
export type WorkflowStatus = z.infer<typeof workflowStatusSchema>;

// ✅ Backend uses Prisma types
import { prisma } from "@/shared/prisma";
const run = await prisma.workflowRun.findUnique({ where: { id } });

// ✅ Frontend defines custom interfaces
export interface WorkflowRun {
  id: string;
  status: WorkflowStatus;
}
```

### Type Conventions

✅ **DO** - Database fields use `null`:

```typescript
interface Project {
  description: string | null; // Prisma field
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jnarowski/agentcmd](https://github.com/jnarowski/agentcmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
