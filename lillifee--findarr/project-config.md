---
trigger: always_on
description: Findarr is a full-stack TypeScript monorepo for discovering movies and TV shows. It integrates with TMDB (metadata), Radarr/Sonarr (media requests), and Jellyfin (library availability). Users can search, vote, and request media; the app tracks download and availability status via background schedulers.
---

# Copilot Instructions for Findarr

## Project Overview

Findarr is a full-stack TypeScript monorepo for discovering movies and TV shows. It integrates with TMDB (metadata), Radarr/Sonarr (media requests), and Jellyfin (library availability). Users can search, vote, and request media; the app tracks download and availability status via background schedulers.

---

## Monorepo Structure

pnpm workspaces monorepo (`pnpm-workspace.yaml`). Build order is topological: `shared` → `web` + `api`.

```
findarr/
├── packages/shared/      # @findarr/shared — Zod schemas, DB schema, types, helpers
├── apps/api/             # @findarr/api — Fastify backend, SQLite via Drizzle ORM
├── apps/web/             # @findarr/web — React + Vite frontend
├── vite.config.ts        # Root Vite+ config — shared fmt + lint settings for all packages
└── package.json          # Root scripts (build, check, test, db:generate, …)
```

---

## Shared Package (`@findarr/shared`)

Single source of truth for types, schemas, and helpers shared across api and web. Split into **subpath modules** (no root barrel) — always import from the specific subpath, never from `@findarr/shared` directly. Each module co-locates its Zod schemas with the types inferred from them. Subpath names mirror the `apps/api/src` feature modules.

| Subpath                       | Source               | Contents                                                                                                                                                                                                                                              |
| ----------------------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `@findarr/shared/db`          | `src/db.ts`          | Drizzle ORM table definitions + relations (SQLite) and the `Db*` row types inferred from them (plus `SeasonRecord`, the `media.seasons` JSON column shape). When modifying the DB schema, update this file then run `pnpm run db:generate` from root. |
| `@findarr/shared/media`       | `src/media.ts`       | Media domain types (Movie, TVShow, details, scoring, response wrappers) + DB-derived media composites (`MediaRecord`, `MediaUser`, `MediaInteraction`, `MediaInteractionWithUser`, `MediaVotes`).                                                     |
| `@findarr/shared/catalog`     | `src/catalog.ts`     | Catalog/browse request schemas + inferred types (search, discover, popular, details, genres).                                                                                                                                                         |
| `@findarr/shared/settings`    | `src/settings.ts`    | User settings + integration settings schemas/types (TMDB, Radarr, Sonarr, Jellyfin).                                                                                                                                                                  |
| `@findarr/shared/preferences` | `src/preferences.ts` | DB-derived user preference types (`UserGenrePreference`, `UserKeywordPreference`).                                                                                                                                                                    |
| `@findarr/shared/auth`        | `src/auth.ts`        | Auth + user schemas/types (login, password, user CRUD) and the `User` entity type (`Omit<DbUser, 'passwordHash'>`).                                                                                                                                   |
| `@findarr/shared/interaction` | `src/interaction.ts` | Like/dislike interaction schemas/types.                                                                                                                                                                                                               |
| `@findarr/shared/scheduler`   | `src/scheduler.ts`   | Scheduler config/state types + name/param schemas.                                                                                                                                                                                                    |
| `@findarr/shared/constants`   | `src/constants.ts`   | Region groups, unified genres, and their keys/types.                                                                                                                                                                                                  |
| `@findarr/shared/utils`       | `src/utils.ts`       | Pure utility helpers (`isDefined`, `getErrorMessage`, object helpers).                                                                                                                                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lillifee/findarr](https://github.com/Lillifee/findarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
