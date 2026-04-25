---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT: AI Assistant Rules

**ALWAYS check `.agent/` directory FIRST before starting any work:**

- `.agent/rules.md` - Project-specific rules that OVERRIDE default behavior
- `.agent/docs/*-learnings.md` - Workspace-specific insights and patterns

These files contain critical information about how to work with this codebase effectively.

## Project Overview

**template** is a modern full-stack monorepo template built with TanStack Start, Convex, and Three.js. It provides a solid foundation for building real-time web applications with 3D graphics capabilities.

## Architecture

### Monorepo Structure

- `apps/web/` - React web app (TanStack Start)
- `apps/convex/` - Convex real-time backend
- `packages/types/` - Shared TypeScript interfaces (@template/types)
- `packages/utils/` - Shared utility functions (@template/utils)
- `terraform/` - Infrastructure as code
- `.github/workflows/` - CI/CD pipelines
- `.agent/` - AI assistant documentation and learnings

### Tech Stack

- **Frontend**: TanStack Start, shadcn/ui, Tailwind CSS v4, TanStack Query/Router, Three.js, Framer Motion
- **Backend**: Convex (real-time DB + serverless functions), Clerk (auth)
- **Infrastructure**: Cloudflare Workers, Terraform, ngrok (local webhooks)
- **Development**: Bun, Nx, Vitest, TypeScript, ESLint, Prettier

## Development Commands

### Core Commands

```bash
bun run dev           # Start full dev environment (frontend + backend)
bun run dev:frontend  # Start frontend only
bun run dev:backend   # Start backend only
bun run build         # Build all projects
bun run test          # Run all tests
bun run typecheck     # Type check all projects
bun run lint          # Lint all projects
```

### Quality Checks

```bash
bun run quality       # Run typecheck + lint + format check
bun run quality:fix   # Run typecheck + lint fix + format
```

### Nx Commands

```bash
bun nx show projects                    # List all projects
bun nx show project <project>           # Show project details
bun nx <task> <project>                 # Run task for specific project
bun nx run-many --target=<task>         # Run task for all projects
bun nx reset                            # Clear Nx cache
```

### Testing

- **Framework**: Vitest with Happy DOM
- **Frontend**: @testing-library/react for component testing
- **Backend**: convex-test for Convex function testing
- **Run single test**: `bun nx test <project> -- <test-file>`

## Code Conventions & Style Guide

### File Naming

- **ALWAYS use `kebab-case`** for file names (e.g., `user-profile.tsx`, `search-utils.ts`)
- Test files: `[name].test.ts` or `[name].test.tsx`
- NO underscores, NO camelCase in file names

### Naming Conventions

#### TypeScript/JavaScript

- **camelCase** for:
  - Variable names: `const userName = 'John'`
  - Function names (non-components): `function getUserData() {}`
  - Object properties: `{ firstName: 'John', lastName: 'Doe' }`
- **PascalCase** for:
  - React components: `function UserProfile() {}`
  - Classes: `class UserService {}`
  - Types/Interfaces: `type UserData = {}`, `interface UserProfile {}`
  - Enums: `enum UserRole {}`

- **UPPER_SNAKE_CASE** for:
  - Constants: `const MAX_RETRIES = 3`
  - Environment variables: `process.env.DATABASE_URL`

### UI Design Style

- **Lowercase text** throughout the UI (buttons, labels, headers)
  - Button: "save changes" NOT "Save Changes"
  - Headers: "user profile" NOT "User Profile"
  - Labels: "email address" NOT "Email Address"
- Exception: Proper nouns and user-generated content maintain their casing

### Import Rules

- **shadcn/ui components** can ONLY be imported in `apps/web/` directory
  - Reason: Only `apps/web/components.json` exists
  - Example: `import { Button } from '@/components/ui/button'`
  - NEVER attempt to use shadcn imports in other workspaces

### Code Style

- **Indentation**: 2 spaces for TypeScript/JavaScript/JSON, 4 spaces for Python/Rust
- **No comments** unless explicitly requested
- **ESLint Error Handling**: For unused variables that would create TypeScript or logic issues if prefixed with underscore, use `// eslint-disable-next-line @typescript-eslint/no-unused-vars` instead
- Prefer existing utility functions over creating new ones

### Import Patterns

```typescript
// From web app
import { api } from '@template/convex';
import type { User, DataType } from '@template/types';
import { utilityFunction } from '@template/utils';
import { cn } from '@/utils/tailwind-utils';

// From backend
import { query, mutation } from './_generated/server';
import { v } from 'convex/values';
import type { User } from '@template/types';
```

## Convex Backend Development

### Database Queries

Prefer Convex indexes over filters for performance:

```typescript
// Good - using index
const items = await ctx.db
  .query('items')
  .withIndex('by_user', (q) => q.eq('userId', userId))
  .collect();

// Avoid - using filter
const items = await ctx.db
  .query('items')
  .filter((q) => q.eq(q.field('userId'), userId))
  .collect();
```

### Function Types

- **Queries**: Read-only data fetching
- **Mutations**: Data modifications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jjjjjjjjjjjjjjjjacob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
