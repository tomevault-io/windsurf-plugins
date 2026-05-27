---
trigger: always_on
description: This document provides essential information for AI coding agents working on the HackClub.tv codebase.
---

# Agent Guidelines for HackClub.tv

This document provides essential information for AI coding agents working on the HackClub.tv codebase.

## Project Overview

HackClub.tv is a live streaming platform built with Next.js 16, Prisma, and Turbo monorepo architecture.

- **Monorepo**: Turborepo with pnpm workspaces
- **Apps**: web (Next.js), chat (Hono), docs
- **Packages**: db (Prisma), auth (Lucia), hono-ws, sdk
- **Package Manager**: pnpm 10.6.5

## Build, Lint, and Test Commands

### Root Level Commands

```bash
pnpm install            # Install all dependencies
pnpm build              # Build all apps and packages (uses Turbo)
pnpm dev                # Start all apps in dev mode (uses Turbo)
pnpm lint               # Lint all apps (uses Turbo)
```

### Database Commands

```bash
pnpm db:migrate         # Run Prisma migrations in dev
pnpm prisma             # Run any Prisma command in db package
```

### App-Specific Commands

```bash
# Web app (Next.js)
pnpm --filter=@hctv/web dev           # Start Next.js dev server
pnpm --filter=@hctv/web build         # Build Next.js app
pnpm --filter=@hctv/web lint          # Lint web app
pnpm --filter=@hctv/web check-types   # Type check (tsc --noEmit)
pnpm --filter=@hctv/web ui:add        # Add shadcn components

# Chat app (Hono)
pnpm --filter=@hctv/chat dev          # Start chat server with watch
pnpm --filter=@hctv/chat build        # Build chat server

# DB package (Prisma)
pnpm --filter=@hctv/db db:generate    # Generate Prisma client
pnpm --filter=@hctv/db db:migrate     # Run migrations
pnpm --filter=@hctv/db build          # Generate client and build
```

### Running Single Tests

This project does not currently have a test suite configured. When adding tests:

- Use Vitest or Jest for unit tests
- Use Playwright for E2E tests (recommended for Next.js)
- Follow the pattern: `pnpm test -- <test-file-path>`

### Docker Commands

```bash
pnpm docker:web         # Build web app Docker image
pnpm docker:chat        # Build chat app Docker image
pnpm r:rtmp             # Restart RTMP server
```

## Code Style Guidelines

### Formatting

- **Formatter**: Prettier (`.prettierrc.json`)
- **Indentation**: 2 spaces (no tabs)
- **Line Width**: 100 characters
- **Quotes**: Single quotes
- **Semicolons**: Required
- **Trailing Commas**: ES5 style
- **Linter**: ESLint with Next.js rules (`next/core-web-vitals`)

### Import Organization

Imports should be ordered as follows (no blank lines between groups):

1. React/Next.js core imports
2. Third-party libraries
3. Internal components (`@/components`)
4. Internal utilities/libs (`@/lib`)
5. Package imports (`@hctv/*`)
6. Type imports (use `import type` keyword)
7. Relative imports

```typescript
import { useState, useEffect } from 'react';
import { Send } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { validateRequest } from '@/lib/auth/validate';
import { prisma } from '@hctv/db';
import type { User, Channel } from '@hctv/db';
import { helper } from './utils';
```

### TypeScript Usage

- **Strict mode**: Enabled
- **Type over interface**: Use `type` for unions/intersections, `interface` for object shapes
- **No implicit any**: Always type your variables
- **Type imports**: Use `import type` for type-only imports
- **Prisma types**: Import from `@hctv/db` and use type composition

```typescript
// Interfaces for props and object shapes
interface ChatMessage {
  user?: User;
  message: string;
  type: 'message' | 'systemMsg';
}

// Type aliases for complex types
type FormFieldConfig = {
  name: string;
  label?: string;
};

// Prisma type composition
type StreamWithChannel = StreamInfo & { channel: Channel };
```

### Naming Conventions

**Files:**

- React components: `PascalCase.tsx` (e.g., `ChatPanel.tsx`)
- Utilities/helpers: `camelCase.ts` (e.g., `validate.ts`)
- Next.js pages: `page.tsx`, `route.ts`, `layout.tsx`
- Client components: `page.client.tsx`

**Variables & Functions:**

- Components: `PascalCase`
- Functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Booleans: Use `is`, `has`, `should` prefixes
- Refs: `Ref` suffix (e.g., `socketRef`)

```typescript
const MESSAGE_HISTORY_SIZE = 15;
const isFollowing = await checkFollowing();
const socketRef = useRef<WebSocket | null>(null);
```

### Error Handling

**API Routes:**

- Return `Response` objects with appropriate status codes
- Use descriptive error messages
- Status codes: 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found)

```typescript
if (!user) {
  return new Response('Unauthorized', { status: 401 });
}
```

**Server Actions:**

- Return objects with `success` boolean and `error` or `data` fields
- Use `zodVerify` helper for validation

```typescript
const zod = await zodVerify(schema, formData);
if (!zod.success) {
  return { success: false, error: zod.error };
}
return { success: true, data: result };
```

**Client-side:**

- Use try-catch for async operations
- Use toast notifications (sonner) for user feedback
- Log errors with `console.error`

### Async Patterns

- **Prefer**: async/await over Promise chains
- **Parallel operations**: Use `Promise.all()`
- **No .then() chaining**: Except in utility functions like fetchers

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SrIzan10/hctv](https://github.com/SrIzan10/hctv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
