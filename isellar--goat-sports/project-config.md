---
trigger: always_on
description: You are working on GOAT Sports, a next-generation fantasy sports platform built with:
---

# GOAT Sports - Cursor AI Rules

## Project Context
You are working on GOAT Sports, a next-generation fantasy sports platform built with:
- **Framework**: Next.js 15 (App Router) with TypeScript
- **Database**: PostgreSQL via Drizzle ORM
- **UI**: shadcn/ui components (Radix UI primitives) with Tailwind CSS
- **Package Manager**: Bun (preferred) or npm
- **State Management**: React Query (TanStack Query) for server state

## Beads Issue Tracking

This project uses **beads** (`bd`) for persistent issue tracking across sessions.

### Core Commands
```bash
bd ready                 # Find available work (no blockers)
bd show <id>             # View issue details with dependencies
bd list --status=open    # All open issues
bd create --title="..." --type=task|bug|feature --priority=2
bd update <id> --status=in_progress  # Claim work
bd close <id>            # Complete work
bd close <id1> <id2>     # Close multiple issues efficiently
bd sync                  # Sync beads changes with git
bd dep add <issue> <depends-on>  # Add dependency
bd blocked               # Show all blocked issues
```

### When to Use Beads
- Multi-session work that needs persistence
- Complex tasks with dependencies or blockers
- Strategic work that survives conversation compaction
- Discovered work during implementation
- Parallel work streams

### Priority Levels
Use numeric priorities: `0-4` or `P0-P4`
- **P0/0**: Critical (production down, data loss)
- **P1/1**: High (blocks key features)
- **P2/2**: Medium (normal work, default)
- **P3/3**: Low (nice to have)
- **P4/4**: Backlog (future consideration)

### Session Completion - MANDATORY

**Before ending ANY session, complete ALL steps:**

```bash
# 1. Create issues for remaining work
bd create --title="..." --type=task --priority=2

# 2. Run quality gates (if code changed)
bun run type-check && bun run lint && bun run test

# 3. Update beads status
bd close <id1> <id2>  # Close completed work
bd update <id> --notes="Current status..."  # Update in-progress

# 4. Commit code changes
git add <files>
git commit -m "Description

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

# 5. Push to remote (MANDATORY - work is NOT done until this succeeds)
git pull --rebase
bd sync
git push
git status  # MUST show "up to date with origin"
```

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - stranded local work is lost work
- If push fails, resolve conflicts and retry until success
- Create issues for ALL remaining work before ending session

## Code Style & Conventions

### TypeScript
- Use strict TypeScript with full type inference
- Leverage Drizzle's type inference from schema - never manually type database results
- Use `type` for type aliases, `interface` for object shapes that may be extended
- Prefer explicit return types for functions that return complex types
- Use `as const` for literal types when needed

### File Organization
- **API Routes**: `app/api/[resource]/route.ts` - use Next.js 15 App Router conventions
- **Components**: `components/[feature]/ComponentName.tsx` - feature-based organization
- **UI Components**: `components/ui/` - shadcn/ui components (don't modify these directly)
- **Database**: `lib/db/schema.ts` - single source of truth for all database schemas
- **Utils**: `lib/utils/` - shared utility functions
- **Types**: Inline with components/API routes, or in `lib/types/` if shared
- **Documentation**: All documentation files (except README.md) go in `docs/` folder

### Naming Conventions
- **Components**: PascalCase (e.g., `PlayerCard.tsx`)
- **Functions**: camelCase (e.g., `calculateFantasyPoints`)
- **Constants**: UPPER_SNAKE_CASE for true constants, camelCase for exported configs
- **Database tables**: plural, lowercase (e.g., `players`, `teams`)
- **Database columns**: camelCase in schema, snake_case in DB (Drizzle handles conversion)

### Database Patterns
- **Always use Drizzle ORM** - never raw SQL unless absolutely necessary
- **Batch queries** - avoid N+1 queries, use `inArray()` for batch fetching
- **Type inference** - use `typeof schema.players.$inferSelect` for types
- **Relations** - define in `schema.ts` using Drizzle relations API
- **Migrations** - always generate via `bun run db:generate` after schema changes

### API Route Patterns
```typescript
// Standard API route structure
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';
import { players } from '@/lib/db/schema';
import { eq, and, or } from 'drizzle-orm';

export async function GET(request: NextRequest) {
  try {
    if (!db) {
      return NextResponse.json(
        { error: 'Database not configured' },
        { status: 503 }
      );
    }
    // Implementation
    return NextResponse.json({ data });
  } catch (error) {
    console.error('Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Component Patterns
- **Server Components by default** - use `'use client'` only when needed (interactivity, hooks, browser APIs)
- **Type props explicitly** - use TypeScript interfaces for component props
- **Use shadcn/ui components** - import from `@/components/ui/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isellar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
