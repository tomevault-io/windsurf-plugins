---
trigger: always_on
description: Flashpoint Web: self-hosted web app for browsing/playing Flashpoint Archive
---

# CLAUDE.md

## Project Overview

Flashpoint Web: self-hosted web app for browsing/playing Flashpoint Archive
games.

- **Backend** (Express/TypeScript, port 3100): REST API + game content via
  `/game-proxy/*`, `/game-zip/*`
- **Frontend** (Vite/React/TypeScript, port 5173): TanStack Query, Zustand,
  Tailwind, Ruffle
- **Databases**: `flashpoint.sqlite` (read-only, never write) + `user.db` (app
  data)

## Commands

```bash
npm run install:all        # Install all
npm run dev                # Start all services
npm run typecheck          # Type check all
npm run build              # Build all
npm test                   # Run backend tests
```

## Documentation

Docs in `docs/` (100+ files). **Reference docs instead of repeating**:

| Topic            | Path                                                |
| ---------------- | --------------------------------------------------- |
| Architecture     | `docs/02-architecture/system-architecture.md`       |
| API Reference    | `docs/06-api-reference/README.md`                   |
| Common Pitfalls  | `docs/08-development/common-pitfalls.md`            |
| Setup Guide      | `docs/08-development/setup-guide.md`                |
| Environment Vars | `docs/09-deployment/environment-variables.md`       |
| Database Schema  | `docs/12-reference/database-schema-reference.md`    |
| Game Service     | `docs/05-game-service/architecture.md`              |
| Components       | `docs/04-frontend/components/component-overview.md` |

---

## Coding Standards (MANDATORY)

**Follow these rules strictly. Violations will be caught in code review.**

### TypeScript & Types

- **No `any` types** — use `unknown`, generics, or proper interfaces
- **No non-null assertions (`!`)** — use `?? defaultValue` or proper null checks
- **Use `??` not `||`** for defaults — `||` treats `0`, `""`, `false` as falsy
- **Validate `parseInt()`** — always check `isNaN()`, NaN silently passes to SQL
- **Throw `AppError`** not plain `Error` — ensures proper HTTP status codes
- **Define return types** for exported functions — improves type inference
- **Use `readonly` arrays** when mutation isn't needed — `readonly string[]`
- **Prefer `unknown` over `any`** for caught errors — `catch (error: unknown)`

```typescript
// WRONG
const limit = parseInt(req.query.limit) || 10; // NaN becomes 10, but "0" also becomes 10
const user = users.find((u) => u.id === id)!; // Crashes if not found

// CORRECT
const parsed = parseInt(req.query.limit as string, 10);
const limit = isNaN(parsed) ? 10 : parsed;
const user = users.find((u) => u.id === id) ?? null;
```

### Express Backend

- **`asyncHandler()` on ALL async route handlers** — prevents unhandled promise
  rejections
- **Static routes BEFORE parameterized** — `/random` must come before `/:id`
- **Check `res.headersSent`** before error responses — required for
  SSE/streaming
- **Cap query limits** — `Math.min(parsedLimit, 100)` prevents data dumps
- **Add lower bounds too** — `Math.max(1, Math.min(parsedLimit, 100))`
- **Validate query params** — use Zod schemas, check bounds (min/max)
- **Wrap check-then-insert in `db.transaction()`** — prevents TOCTOU race
  conditions
- **Strip port from hostname** — `req.headers.host?.split(':')[0]`
- **Return early on validation failure** — don't continue processing bad input
- **Use `requirePermission()` middleware** — for protected endpoints

```typescript
// WRONG - async handler without wrapper
router.get('/games', async (req, res) => { ... });

// WRONG - parameterized route shadows static
router.get('/:id', ...);
router.get('/random', ...);  // Never reached!

// CORRECT
router.get('/random', asyncHandler(async (req, res) => { ... }));
router.get('/:id', asyncHandler(async (req, res) => { ... }));
```

### Database

- **Never write to `flashpoint.sqlite`** — it's read-only, managed by Flashpoint
  Launcher
- **Use transactions for multi-step operations** — atomic success or rollback
- **Batch large operations** — wrap all batches in single outer transaction
- **Use scalar subqueries or window functions** — avoid separate COUNT queries
- **Build Maps for O(1) lookups** — never use `.find()` in loops over large
  datasets
- **Always use parameterized queries** — never string interpolation for SQL

```typescript
// WRONG - O(N²) complexity
const results = games.map((game) => {
  const favorite = favorites.find((f) => f.gameId === game.id); // Scans entire array
  return { ...game, isFavorite: !!favorite };
});

// CORRECT - O(N) with Map
const favoriteSet = new Set(favorites.map((f) => f.gameId));
const results = games.map((game) => ({
  ...game,
  isFavorite: favoriteSet.has(game.id),
}));

// WRONG - separate COUNT query
const games = db.all('SELECT * FROM game LIMIT ?', [limit]);
const total = db.get('SELECT COUNT(*) as count FROM game').count;

// CORRECT - window function
const games = db.all(
  `
  SELECT *, COUNT(*) OVER() as total_count
  FROM game LIMIT ?
`,
  [limit]
);
```

### React Frontend

- **All API calls through `@/lib/api.ts`** — never raw `fetch()`, bypasses
  auth/refresh
- **Use `@/` imports** — not relative `../` paths
- **Use theme tokens** — `text-muted-foreground` not `text-gray-400`
- **Store callbacks in `useRef`** when used in `useEffect` — prevents infinite

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkraise/flashpoint-web](https://github.com/darkraise/flashpoint-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
