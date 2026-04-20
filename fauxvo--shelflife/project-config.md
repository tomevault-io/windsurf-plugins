---
trigger: always_on
description: Plex library storage manager. Users log in via Plex, see their Overseerr requests, and vote to keep or delete content. Admins see aggregate deletion candidates.
---

# Shelflife

Plex library storage manager. Users log in via Plex, see their Overseerr requests, and vote to keep or delete content. Admins see aggregate deletion candidates.

## Tech Stack

- **Framework**: Next.js 16 (App Router, React 19)
- **Language**: TypeScript (strict mode)
- **Database**: SQLite via Drizzle ORM + better-sqlite3
- **Validation**: Zod schemas
- **Styling**: Tailwind CSS v4
- **Auth**: Plex PIN-based OAuth with JWT sessions
- **Testing**: Vitest with in-memory SQLite
- **Package Manager**: `bun` for install/run, `npm test` for tests

## Project Structure

```
src/
  app/                    # Next.js App Router pages and API routes
    api/
      admin/              # Admin-only endpoints (requireAdmin)
      auth/plex/          # Plex OAuth flow (pin creation, callback)
      media/              # User media endpoints (requireAuth)
      sync/               # Data sync endpoints (requireAdmin)
      health/             # Health check (public)
  components/
    admin/                # Admin dashboard components
    auth/                 # Login button
    media/                # Media cards, grid, voting
    ui/                   # Shared UI (Pagination, MediaTypeBadge, MediaCardSkeleton)
  lib/
    auth/                 # Session management, auth middleware
    db/                   # Drizzle instance, schema, shared query helpers
    services/             # External API clients (Overseerr, Tautulli, Plex)
    validators/           # Zod schemas for request validation
    constants.ts          # Shared UI constants (colors, labels)
  test/                   # Test setup and helpers
  types/                  # TypeScript type definitions
```

## Commands

```bash
bun install              # Install dependencies
bun run dev              # Start dev server
bun run build            # Production build
bun run lint             # ESLint check
bun run lint:fix         # ESLint auto-fix
bun run format           # Prettier format all source files
bun run format:check     # Prettier check formatting
npm test                 # Run all tests (always use npm, not bun)
npm test -- --watch      # Run tests in watch mode
```

## Architecture Principles

### Sync-Based Data Model

- External APIs (Sonarr, Radarr, Seerr/Overseerr, Tautulli, Tracearr) are read-only data sources
- Data is synced into local SQLite -- the app works even if external services are down
- Votes and user data are stored locally, never pushed to external services
- All sync operations go through `src/lib/services/sync.ts`

### Auth Model

- Plex PIN-based OAuth (same flow as Overseerr)
- JWT sessions stored in cookies
- Middleware in `src/proxy.ts` protects all routes except public paths and static assets
- Route handlers use `requireAuth()` or `requireAdmin()` from `src/lib/auth/middleware.ts`
- First user to sign in becomes admin automatically

### Data Access

- The media endpoint supports a `scope` param: `scope=personal` (default) filters to only the current user's requests; `scope=all` shows all users' library items
- Voting is public and transparent — nomination attribution (who nominated), community vote tallies, and voter usernames are visible to all authenticated users in the community view
- Watch status data is scoped to the current user via LEFT JOINs (users see their own watch history, not others')
- Any authenticated user can nominate any library item for deletion/trim — ownership is NOT enforced (supports libraries without request tracking, e.g., Sonarr/Radarr-primary mode where `requestedByPlexId` is often null)
- Non-admin users are rate-limited to `MAX_NOMINATIONS_PER_ROUND` (100) nominations per review round; admins are exempt
- The rate limit check + vote upsert run inside a single synchronous transaction to prevent concurrent requests from exceeding the cap
- Admin review rounds are the governance layer — nominations only exist during active rounds, and admins decide final actions
- The shared `getNominationCondition()` helper in `lib/db/queries.ts` encapsulates the nomination visibility rule: any user's nominations in the active round
- Vote/watched filtering happens in SQL WHERE clauses, not post-query in JS

## Code Conventions

### API Routes

- Always wrap handler body in try/catch with `handleAuthError(error)` in the catch
- Use `requireAuth()` for user endpoints, `requireAdmin()` for admin endpoints
- Validate query params with Zod schemas from `src/lib/validators/schemas.ts`
- Use shared query helpers from `src/lib/db/queries.ts` (`mediaQueryWithJoins`, `mediaCountWithJoins`, `mapMediaItemRow`, `buildPagination`)
- Return consistent pagination shape: `{ items: [...], pagination: { page, limit, total, pages } }`

### Components

- Use shared `<MediaTypeBadge>` for TV/Movie badges
- Use shared `<MediaCardSkeleton>` for loading states
- Use shared `<Pagination>` component (not custom pagination buttons)
- Import colors and labels from `src/lib/constants.ts` (`STATUS_COLORS`, `VOTE_COLORS`, `VOTE_LABELS`)
- Client components must have `"use client"` directive

### Database

- Use Drizzle ORM query builder -- never raw SQL strings (prevents SQL injection)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fauxvo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
