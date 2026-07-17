---
trigger: always_on
description: - Monorepo: pnpm workspaces. Apps in `apps/*`, shared packages in `packages/*`.
---

# AGENTS.md — how to work in this repo

## Stack
- Monorepo: pnpm workspaces. Apps in `apps/*`, shared packages in `packages/*`.
- Mobile: `@tvwatch/mobile` (Expo SDK 54 + Expo Router 6). TypeScript + React Native.
- API: `@tvwatch/api` (NestJS 10 + Prisma 5 + PostgreSQL 16 + Redis 7 + BullMQ).
- Admin: `@tvwatch/admin` (Next.js 14 + Tailwind + Recharts).
- Shared: `@tvwatch/shared` (types/contracts used by both apps, CJS dist).

## Common commands
- Install: `pnpm install`
- Infra: `docker compose up -d` (Postgres, Redis, MinIO)
- DB: `pnpm db:generate`, `pnpm db:migrate`, `pnpm db:seed`
- Dev: `pnpm dev:api`, `pnpm dev:mobile`, `pnpm --filter @tvwatch/admin dev`
- Validate: `pnpm typecheck`, `pnpm lint`, `pnpm test`
- After schema changes: `$env:DATABASE_URL="..."; pnpm --filter @tvwatch/api prisma db push --accept-data-loss; pnpm --filter @tvwatch/api prisma generate`
- Data-migrating schema changes: `prisma db push` only applies DDL diffs — it CANNOT run backfill SQL (it will offer to reset the DB instead). For migrations that transform existing rows (e.g. `20260712195500_episode_voting` re-keying `character_votes.character_name` → `cast_id`), apply the migration SQL directly, then `db push` is a no-op:
  ```powershell
  pnpm --filter @tvwatch/api prisma db execute --file prisma/migrations/<migration>/migration.sql --schema prisma/schema.prisma
  pnpm --filter @tvwatch/api prisma db push   # no-op once the DB matches the schema
  ```

## Required builds after changes
- If any API code, API dependency, Prisma schema, shared backend contract, or API Dockerfile/configuration changes, rebuild and publish the API image from the repository root:
  ```powershell
  docker build --no-cache -t ghcr.io/metalingus/tvwatch-api:latest -f apps/api/Dockerfile .
  docker push ghcr.io/metalingus/tvwatch-api:latest
  ```
- If any web-facing code in `apps/mobile` or a shared package used by the web app changes, rebuild the Expo web export from `apps/mobile`:
  ```powershell
  npx expo export --platform web --output-dir ../app-web
  ```
- Do not claim a build succeeded unless the corresponding command completed successfully. Report any build or push failure with the relevant error.

## Conventions
- Always import shared types from `@tvwatch/shared` — do not duplicate DTOs across apps.
- The Prisma schema (`apps/api/prisma/schema.prisma`) is the source of truth for the data model. Regenerate after edits: `pnpm db:generate`.
- Mobile NEVER calls third-party media APIs directly. All media data flows through the backend, which normalizes + caches external IDs.
- Use snake_case only in DB column names via Prisma `@map`. In code/TS use camelCase.
- Prettier config is at repo root (`.prettierrc.json`). Single quotes, trailing comma all, 100 width.
- Env vars: read via NestJS `ConfigService`. Never hardcode secrets.
- Special seasons (S0, `isSpecial = true`) are excluded from ALL counts, progress, and watch-next queries.
- Aired episodes only: unaired episodes (`airDate > now`) are excluded from progress bars and watch-next counts.
- The app reads `POSTGRES_*` and `REDIS_*` env vars directly (passwords with special chars are fine). `DATABASE_URL` is only for the Prisma CLI — URL-encode special chars there.

## Adding a backend module
1. Add models to `schema.prisma` + run `pnpm db:generate` and a migration.
2. Create `module`/`service`/`controller`/`dto` under `apps/api/src/<module>`.
3. Use `@CurrentUser()` decorator + `JwtAuthGuard` for authenticated routes.
4. Export the module from `AppModule`.

## Adding a mobile screen
1. Add route under `apps/mobile/app/` (Expo Router file-based).
2. Fetch via `apps/mobile/api/client.ts` (`api.get`/`api.post`) which injects the JWT.
3. Use the shared theme (`apps/mobile/theme/theme.ts`) + component system in `apps/mobile/components`.
4. Respect dark theme + safe areas.
5. Icons: `@expo/vector-icons` (Ionicons).
6. Images: `expo-image` `Image` (NOT React Native Image, NOT `PosterImage` for search results — use `expo-image` directly with `contentFit="cover"`).

## Localization and theme requirements
- All user-facing text must use the existing translation/i18n system. Do not introduce hardcoded UI strings when a translation key should be used.
- When adding or changing user-facing copy, add or update the key in every supported locale. Check for missing, stale, or fallback-only translations before finishing.
- Reuse existing translation keys when their meaning matches; keep key names consistent and descriptive.
- All colors, spacing, typography, radii, shadows, and other visual values must come from the shared theme/design tokens whenever a token exists. Do not add unexplained hardcoded visual values.
- Components must work with the supported light, dark, and system-selected themes. Verify contrast and state styling in both light and dark modes.

## Mobile grid pattern (IMPORTANT)
- NEVER use `FlatList numColumns` or `flexWrap` + `gap` — both cause bugs on Android.
- Use chunked rows: split items into arrays of N, render each row as a `<View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>`, add invisible spacer Views for incomplete rows.
- `PosterCard` accepts a `style` prop — pass `{ marginRight: 0 }` inside grids.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Metalingus/tvwatchtime](https://github.com/Metalingus/tvwatchtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
