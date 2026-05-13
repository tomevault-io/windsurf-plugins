---
trigger: always_on
description: pnpm install && pnpm run dev  # API (3001) + Web (3000)
---

# CLAUDE.md

## Quick Start

```bash
pnpm install && pnpm run dev  # API (3001) + Web (3000)
```

## Critical Rules

These cause bugs if ignored:

1. **API Proxy**: Frontend calls `/api/*` via Next.js rewrites to backend. NEVER use `localhost:3001` directly from frontend code.
2. **Ownership**: Always include `userId: request.currentUser!.id` in Prisma queries for user-owned resources. Omitting this is a security vulnerability.
3. **Encryption**: All API keys must be encrypted with `app.encryptor.encrypt()` before storage. Store both `value` and `iv`.
4. **Auth Check**: Protected routes use preHandler hook checking `request.currentUser?.id`. Every route plugin must add this hook.
5. **Validation**: Use `validateRequest()` from `lib/utils/validate.ts` for request body parsing. Never use `request.body as Type`.
6. **Incognito Mode**: Any component displaying sensitive data (titles, usernames, URLs, instance names) must use `useIncognitoMode()` hook from `contexts/IncognitoContext.tsx` and anonymize with functions from `lib/incognito.ts`. This includes API response text that embeds instance names (e.g., Pulse titles like `"Label: message"` — split and anonymize both parts). Tests rendering such components need `<IncognitoProvider>` wrapper.
7. **Query Invalidation**: Always invalidate relevant React Query keys after mutations.
8. **Session Invalidation**: After credential changes (password, passkey deletion), call `invalidateAllUserSessions(userId, exceptToken)`.

## Key Paths

```
apps/api/src/routes/       # API route handlers
apps/api/src/lib/          # Shared backend logic (auth/, arr/, utils/, hunting/, queue-cleaner/)
apps/api/prisma/schema.prisma  # Database schema (source of truth)
apps/web/app/              # Next.js pages
apps/web/src/features/     # Feature-specific components, organized by domain
apps/web/src/hooks/api/    # React Query hooks
apps/web/src/lib/api-client/   # API client modules
apps/web/src/lib/theme-gradients.ts  # Color system source of truth
apps/web/src/components/layout/premium-components.tsx  # Reusable UI components
packages/shared/src/types/ # Shared Zod schemas + TypeScript types
```

## Architecture

**Monorepo**: `apps/api` (Fastify 4), `apps/web` (Next.js 16 App Router), `packages/shared` (Zod types). pnpm 10+ with Turbo.

**Stack**: Fastify + Prisma + Zod backend, Next.js + React 18 + TanStack Query frontend, TailwindCSS + shadcn/ui, SQLite default (PostgreSQL supported).

**Design**: Single-admin, self-hosted. Session-based auth (NOT JWT). Three auth methods: Password, OIDC, Passkeys.

**Services**: Sonarr, Radarr, Prowlarr, Lidarr, Readarr, Plex, Tautulli, Seerr (Jellyseerr/Overseerr).

**Fastify Decorations** (available in route handlers):
- `request.currentUser` / `request.sessionToken` — populated by auth preHandler
- `app.prisma` — Prisma client
- `app.encryptor` — AES-256-GCM encryption (`.encrypt()` / `.decrypt()`)
- `app.sessionService` — session create/invalidate/cookie management
- `app.config` — environment config

## Code Style & Conventions

- **Linting**: Biome for API + shared packages, ESLint for web (React-specific rules). Biome for formatting across all packages. Run `pnpm run lint` / `pnpm run format`
- **TypeScript**: Strict mode, `noUncheckedIndexedAccess: true`
- **Imports**: Feature modules use relative imports. Base UI components use `@/` alias
- **Components**: Default to Server Components; add `"use client"` when using hooks/interactivity
- **Theming**: Use `useThemeGradient()` hook for theme colors. Never use the old 2-line pattern (`useColorTheme` + `THEME_GRADIENTS[colorTheme]`). See `@docs/THEMING.md` for full reference
- **Colors**: Never hardcode colors. Use `getServiceGradient()` for runtime service lookups, `SEMANTIC_COLORS` for status, `BRAND_COLORS` for external services
- **Z-Index**: Use semantic classes (`z-modal`, `z-toast`, `z-dropdown`), never arbitrary `z-[9999]`
- **Premium Components**: Check `premium-components.tsx` before creating custom UI (has GlassmorphicCard, ServiceBadge, StatusBadge, PremiumTabs, PremiumTable, GradientButton, etc.)
- **Error messages**: Use `getErrorMessage()` from `lib/utils/error-message.ts` instead of `error instanceof Error ? error.message : ...`
- **Logging**: Use `request.log` or `app.log` (pino), never `console.log` in production code
- **Query keys**: All React Query keys must be defined in `lib/query-keys.ts`. Never use inline string arrays. Import from the centralized file.
- **Polling**: Use named constants from `lib/polling-intervals.ts` (POLLING_REALTIME, POLLING_ACTIVE, POLLING_STANDARD, POLLING_STATS, POLLING_BACKGROUND, POLLING_FAST). Never hardcode interval numbers.
- **Refresh pattern**: Use `useRefreshState()` hook from `hooks/useRefreshState.ts` for "refresh with animation" buttons. Never manually manage isRefreshing + setTimeout.
- **No broad refactors**: Do not refactor code unrelated to the current task. Do not "clean up" files you're passing through. Only change what's directly required.
- **Impact summaries**: When completing a task, summarize: what changed, why, files affected, risks, and validation performed.

## Adding Features

**New API Route:**
1. Create `apps/api/src/routes/<domain>.ts`
2. Register in `apps/api/src/server.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kha-kis/arr-dashboard](https://github.com/Kha-kis/arr-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
