---
trigger: always_on
description: Besidka is an open-source AI chat application that runs on Cloudflare Workers. Users bring their own API keys for LLM providers (Google, OpenAI) and pay for what they use.
---

## Project Overview

Besidka is an open-source AI chat application that runs on Cloudflare Workers. Users bring their own API keys for LLM providers (Google, OpenAI) and pay for what they use.

## Package Manager

Use **pnpm** exclusively for this project. Do not use npm, npx, bun, bunx, or other package managers.

## Commands

```bash
# Development
pnpm run dev              # Start dev server with .dev.vars environment
pnpm run build            # Build for production (Cloudflare Workers)
pnpm run preview          # Build and run with wrangler locally

# Type checking & Linting
pnpm run typecheck        # Run Nuxt type checking
pnpm run lint             # Run ESLint
pnpm run format           # Run ESLint with --fix

# Testing
pnpm run test             # Run Vitest in watch mode
pnpm vitest run           # Run tests once
pnpm vitest run path/to/file.test.ts  # Run specific test file

# Database (Drizzle + D1)
pnpm run db:generate      # Generate migrations from schema changes
pnpm run db:migrate       # Apply migrations
pnpm run db:studio        # Open Drizzle Studio
pnpm run db:reset         # Reset local D1 database and regenerate

# Cloudflare
pnpm run cf-typegen       # Generate Cloudflare env types
pnpm run deploy           # Build and deploy to Cloudflare Workers
```

## Architecture

### Directory Structure

- `app/` - Frontend Nuxt application (Vue 3 components, composables, pages)
- `server/` - Backend Nitro server (API routes, database, utilities)
- `shared/` - Code shared between client and server (types, utility functions)
- `providers/` - LLM provider configurations (Google, OpenAI models)

### Project Docs

- `docs/files.md` - Files functionality, upload flow, and Workers constraints

### Tech Stack

- **Framework**: Nuxt 4 with Nitro server preset for Cloudflare Workers
- **Database**: Cloudflare D1 (SQLite) via Drizzle ORM
- **Storage**: Cloudflare KV for caching, R2 for file storage
- **Auth**: Better Auth with email/password, Google, GitHub OAuth
- **AI**: Vercel AI SDK with resumable streams
- **UI**: Tailwind CSS v4 + DaisyUI v5 (see [DaisyUI Reference](#daisyui-reference))

### Key Patterns

**Shared types import**: Always use the `#shared` alias for importing types and utilities from the `shared/` directory. Never use relative imports.
  ```typescript
  // Correct - use #shared alias
  import type { User } from '#shared/types/auth.d'
  import type { Chat } from '#shared/types/chats.d'
  import type { Providers, Provider, Model } from '#shared/types/providers.d'
  import { getModel } from '#shared/utils/model'

  // Wrong - relative imports
  import type { User } from '../../shared/types/auth.d'
  import { getModel } from '../shared/utils/model'
  ```

**Server utilities auto-import**: Functions in `server/utils/` are auto-imported. Key utilities:
- `useDb()` - Get Drizzle D1 database instance
- `useKV()` - Get Cloudflare KV instance
- `useServerAuth()` - Get Better Auth instance
- `useUserSession()` - Get current user session

**Server utility imports**: When a server utility is not auto-imported (e.g., nested helpers), use the `~~/` alias. Never use relative imports in server code.
  ```typescript
  // Correct - use ~~ alias
  import { getReasoningStepsCount } from '~~/server/utils/chats/test/steps-count'

  // Wrong - relative imports
  import { getReasoningStepsCount } from './utils'
  import { getReasoningStepsCount } from '../../utils/chats/test/steps-count'
  ```

**Database schema**: Defined in `server/db/schemas/*.ts`, exported from `server/db/schema.ts`. Uses snake_case column naming.

**API routes**: Located in `server/api/v1/`. Chat endpoints stream AI responses.

**Composables**: Frontend state/logic in `app/composables/`. The `useChat()` composable manages chat state with AI SDK.

**Environment**: Runtime config in `nuxt.config.ts`. Secrets go in `.dev.vars` (gitignored), with example in `.dev.vars.example`.

### Cloudflare Bindings

Configured in `wrangler.jsonc`:
- `DB` - D1 database binding
- `KV` - KV namespace binding
- `R2_BUCKET` - R2 storage bucket
- `IMAGES` - Cloudflare Images binding

**Binding access patterns**:
```ts
// @ts-ignore
import { env } from 'cloudflare:workers'
```

- For D1 and KV: use `const { KV } = env`
- For R2: use `const { R2_BUCKET } = env` 

This is intentional and correct, do NOT change to `event.context.cloudflare.env`

**Simultaneous connection limit**: Workers can only have **6 simultaneous connections** to external services (R2, KV, fetch). Operations beyond 6 are queued. If queued operations wait too long, the Worker hangs with "script will never generate a response" error.

```typescript
// Wrong - spawns many parallel connections, causes hang with 7+ files
await Promise.allSettled(
  keys.map(async (key) => {
    await storage.delete(key)  // Each is a separate connection
  }),
)

// Correct - R2 supports batch delete (single connection)
await storage.delete(keys)

// Correct - sequential operations for KV (no batch delete API)
for (const key of keys) {
  await kv.delete(key)
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [besidka/besidka](https://github.com/besidka/besidka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
