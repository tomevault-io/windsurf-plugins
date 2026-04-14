---
trigger: always_on
description: AutoNode is a Next.js-based AI workflow automation platform that allows users to create and execute visual workflows with various node types (triggers, HTTP requests, AI actions).
---

# AutoNode - AI Workflow Automation Platform

AutoNode is a Next.js-based AI workflow automation platform that allows users to create and execute visual workflows with various node types (triggers, HTTP requests, AI actions).

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Better Auth + Polar (payments)
- **API Layer**: tRPC
- **Background Jobs**: Inngest
- **State Management**: Jotai (atoms), React Query
- **UI**: Tailwind CSS v4, shadcn/ui, Radix UI
- **Workflow Editor**: React Flow (@xyflow/react)

---

## Build & Development Commands

This project uses [Task](https://taskfile.dev/) for running common commands. Make sure Task is installed, then use:

```bash
# Start database and dev server
task start

# Stop everything
task stop

# Start dev server only (Next.js + Inngest)
task dev

# Build for production
task build

# Run linter
task lint

# Database commands
task db:up      # Start Postgres container
task db:down    # Stop Postgres container
task db:push    # Push schema to database
task db:generate  # Generate migrations
task db:migrate  # Apply migrations
task db:studio  # Open Drizzle Studio
task db:reset   # Reset database (removes volumes)
```

Or use pnpm commands directly (if Task not available):

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Run linter
pnpm lint

# Run Inngest dev server (for background jobs)
pnpm inngest:dev

# Run all dev processes (Next.js + Inngest)
pnpm dev:all
```

### Running Tests

This project uses Vitest for testing. Run tests with:

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run a single test file
pnpm test -- path/to/test-file.test.ts

# Run tests matching a pattern
pnpm test -- --grep "workflow"
```

---

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Authentication pages (sign-in, sign-up)
│   ├── (dashboard)/       # Authenticated dashboard pages
│   ├── (marketing)/       # Public marketing pages
│   └── api/               # API routes (trpc, inngest, auth)
├── components/            # Shared UI components
│   ├── ui/               # shadcn/ui components
│   ├── react-flow/       # React Flow node components
│   └── dashboard/        # Dashboard-specific components
├── db/                   # Database layer
│   ├── schema/           # Drizzle schema definitions
│   ├── relations.ts      # Schema relations
│   └── index.ts          # DB client export
├── features/             # Feature-based modules
│   ├── credentials/       # API credentials management
│   ├── executions/       # Workflow execution logic
│   ├── triggers/         # Workflow triggers (manual, cron)
│   └── workflows/        # Workflow CRUD operations
├── hooks/                # Custom React hooks
├── inngest/              # Inngest functions (background jobs)
│   ├── functions/        # Individual Inngest functions
│   └── channels/        # Event channels
├── lib/                  # Utilities and config
│   ├── auth.ts           # Better Auth configuration
│   ├── polar.ts          # Polar payment client
│   └── utils.ts          # cn() utility (clsx + tailwind-merge)
├── services/             # External service integrations
└── trpc/                 # tRPC routers and procedures
    └── routers/          # API route handlers
```

---

## Code Style Guidelines

### Imports

- Use the `@/` alias for src-relative imports (configured in tsconfig.json)
- Order imports: external libs → internal imports → relative imports
- Use explicit imports over barrel exports where it improves readability

```typescript
// Good
import { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { db } from "@/db";
import { workflows } from "@/db/schema";
import { cn } from "@/lib/utils";

// Bad
import { db, workflows } from "@/db"; // Avoid barrel imports
```

### Naming Conventions

- **Files**: kebab-case for utilities, PascalCase for components and types
- **Functions/variables**: camelCase
- **Database columns**: snake_case
- **React components**: PascalCase
- **Types**: PascalCase with `Type` suffix (e.g., `WorkflowType`)

```typescript
// Component files
WorkflowEditor.tsx;
useWorkflows.ts;
node - selecter.tsx;

// Types
type WorkflowType = typeof workflows.$inferSelect;
type NodeEnumType = typeof nodeType.enumValues;
```

### TypeScript

- Always use explicit types for function parameters and return values
- Use type inference for local variables when type is obvious
- Use Zod for runtime validation (especially in tRPC inputs)

```typescript
// Good
export async function getWorkflow(
  id: number,
  userId: string,
): Promise<WorkflowType | undefined> {
  return db.query.workflows.findFirst({
    where: eq(workflows.id, id),
  });
}

// Zod schema for validation
const createWorkflowSchema = z.object({
  name: z.string().min(1),
  nodes: z.array(nodeSchema),
});
```

### React Patterns

- Use `"use client"` directive for client components
- Use `"use server"` for server actions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ashutsoh1729) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
