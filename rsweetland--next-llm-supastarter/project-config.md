---
trigger: always_on
description: Use these conventions on this project. Edit them when versions are upgraded or conventions change.
---

# LLM Conventions

Use these conventions on this project. Edit them when versions are upgraded or conventions change.

## Development Commands

**Package Manager**: pnpm (not npm)

**Development**:
- `pnpm dev` - Start Next.js development server  
- `pnpm dev:inspect` - Start dev server with Node.js debugger (port 9229)

**Testing**:
- `pnpm test` - Run Vitest tests
- `pnpm test:watch` - Run tests in watch mode (auto-runs on file save)
- `pnpm test:inspect` - Run tests with debugger attached
- `pnpm test:ci` - Run tests with coverage for CI

**Database**:
- `pnpm db:generate` - Generate Drizzle migrations (then manually apply SQL to database)
- `pnpm db:push` - Push schema changes to database
- `pnpm db:reset` - Reset database and push schema
- `pnpm db:seed` - Seed database with initial data
- `pnpm db:reset-seed` - Reset database and seed with initial data

**Build**:
- `pnpm build` - Build for production
- `pnpm start` - Start production server

**Utility**:
- `pnpm copy-context` - Copy full codebase to clipboard using repomix

## Architecture

**Framework**: Next.js 15.3 with App Router
- Auth pages in `(auth-pages)` route group
- Protected routes in `protected/` directory  
- API routes in `app/api/`
- Global middleware in `middleware.ts` for session management

**Database**: Supabase (PostgreSQL) with Drizzle ORM v0.43.1
- Schema definitions in `db/schema.ts`
- Database connection in `db/index.ts` 
- Row Level Security policies defined in schema
- Sample `instruments` table with RLS policy for public read access

**Authentication**: Supabase Auth with SSR
- Client/server utilities in `lib/supabase/`
- Middleware handles session updates automatically
- Auth pages for sign-in/sign-up/forgot-password/reset-password

**Testing**: Vitest with React Testing Library
- Tests co-located with source files (`.test.tsx`, `.test.ts`)
- Setup file: `vitest.setup.ts`
- Environment: jsdom for component testing
- Supports debugging through VS Code with included launch configurations

**UI**: 
- Tailwind CSS for styling
- shadcn/ui components (configured in `components.json`)
- Radix UI primitives
- Lucide React icons
- Next.js themes support

**TypeScript**: Full TypeScript setup
- Path aliases: `@/*` maps to project root
- Strict mode enabled
- Types from Next.js, React, and testing libraries

## Key Directories

- `app/` - Next.js App Router pages and layouts
- `components/` - Reusable UI components  
- `lib/` - Utility functions and Supabase helpers
- `db/` - Database schema and connection
- `drizzle/` - Generated migration files
- `__tests__/` - Test files (also co-located with source)

## Environment Setup

**Prerequisites**:
- Supabase CLI installed (`npm install -g supabase`)
- No other Supabase instances running locally

**Local Development**:
- Default configuration supports local Supabase instance
- Ensure Supabase CLI is available and no conflicting instances are running
- Use `supabase start` to spin up local development database

**Environment Variables**:
Required environment variables (see `.env.example`):
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` 
- `DATABASE_URL`

For testing, copy `.env.test.example` to `.env.test` with test database credentials.

## LLM Development Conventions

**Technology Stack Versions**:
- Database: Supabase (Postgres)
- ORM: Drizzle Version 0.43.1
- Framework: Next.js v15.3
- UI: shadcn, tailwind
- Testing: vitest (`pnpm test`)

**Drizzle ORM Patterns**:

### Schema Definition
```typescript
import { AnyPgColumn } from "drizzle-orm/pg-core";
import { pgEnum, pgTable as table } from "drizzle-orm/pg-core";
import * as t from "drizzle-orm/pg-core";

export const rolesEnum = pgEnum("roles", ["guest", "user", "admin"]);

export const users = table(
  "users",
  {
    id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
    firstName: t.varchar("first_name", { length: 256 }),
    lastName: t.varchar("last_name", { length: 256 }),
    email: t.varchar().notNull(),
    invitee: t.integer().references((): AnyPgColumn => users.id),
    role: rolesEnum().default("guest"),
  },
  (table) => [
    t.uniqueIndex("email_idx").on(table.email)
  ]
);

export const posts = table(
  "posts",
  {
    id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
    slug: t.varchar().$default(() => generateUniqueString(16)),
    title: t.varchar({ length: 256 }),
    ownerId: t.integer("owner_id").references(() => users.id),
  },
  (table) => [
    t.uniqueIndex("slug_idx").on(table.slug),
    t.index("title_idx").on(table.title),
  ]
);
```

### Row Level Security (RLS)

**Basic Policy Definition**:
```typescript
import { sql } from 'drizzle-orm';
import { integer, pgPolicy, pgRole, pgTable } from 'drizzle-orm/pg-core';

export const admin = pgRole('admin');
export const users = pgTable('users', {
	id: integer(),
}, (t) => [
	pgPolicy('policy', {
		as: 'permissive',
		to: admin,
		for: 'delete',
		using: sql``,
		withCheck: sql``,
	}),
]);
```

**Policy Options**:
- `as`: 'permissive' or 'restrictive'
- `to`: Role name (string), 'public', 'current_role', 'current_user', 'session_user', or pgRole object
- `for`: 'all', 'select', 'insert', 'update', 'delete'
- `using`: SQL statement for USING clause

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsweetland/next-llm-supastarter](https://github.com/rsweetland/next-llm-supastarter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
