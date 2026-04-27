---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Chance?

Chance is a social party app played on a single shared phone alongside physical board games. When a game event triggers a "Chance" moment, the active player draws a card from a shared pool. Cards carry dares, drinking mechanics, and prompts. The card pool grows as registered players submit cards across sessions. Guests need no account — they join by entering a display name.

## Terminology

| Term | Definition |
|------|------------|
| **Game Session** | A single instance of play. Created by a registered host; expires after 16 days or when the host leaves. |
| **Player** | An ephemeral game identity scoped to one session. Optionally linked to a User. |
| **User** | A permanent registered account (email + password, invite-code-gated). |
| **CardVersion** | Immutable snapshot of card content. Edits create new versions; draw history references the version drawn. |
| **cardType** | `chanceCard` or `reparationsCard` — set at creation, Card-level (not version-level), admin-only to change post-creation. |
| **card_sharing** | Per-player, per-session setting: `none` / `mine` (default). Controls how a registered player contributes cards to the draw pool. |

## Project Status

This app is **released and in production**. The database is live on real devices. Any schema change requires a migration — never modify `init.ts` alone.

## Monorepo Structure

pnpm workspaces + Turborepo.

```
root/
├── apps/
│   ├── backend/        # Next.js 15 App Router — API + admin portal
│   └── mobile/         # Ionic 8 / Capacitor 8 / React 19 / Vite 6 — targets web + native equally
├── packages/
│   └── core/           # Shared Zod schemas, AppError types, constants
├── turbo.json
└── pnpm-workspace.yaml
```

## Common Commands

```bash
# Install all dependencies
pnpm install

# Run all apps in dev mode
pnpm dev

# Build everything (respects Turbo pipeline: core → apps)
pnpm build

# Run tests
pnpm test

# Lint / typecheck
pnpm lint
pnpm typecheck

# Backend only
pnpm --filter backend dev
pnpm --filter backend db:migrate
pnpm --filter backend db:seed

# Mobile only
pnpm --filter mobile dev        # Vite dev server on port 8100
pnpm --filter mobile build
npx cap sync                    # After build, sync to Android/iOS
npx cap run android
npx cap run ios
```

Turbo pipeline order: `build` → `test`, `lint`, `typecheck`. The `core` package must build before apps.

## Code Style

Prettier config (root `.prettierrc`): double quotes, semicolons, trailing commas (ES5), 4-space indent, 100-char print width.

TypeScript strict mode across all packages.

## Shared Core Package (`packages/core`)

Single source of truth for:

- **Schemas** (`src/schemas/`): Zod schemas for all domain entities — `Card`, `Session`, `Player`, `CardTransfer`, `DrawEvent`, `FilterSettings`, `Vote`, `InvitationCode`, `MutationQueueEntry`. Use these for runtime validation AND TypeScript type inference (via `z.infer<>`). Never duplicate types in the apps.
- **Errors** (`src/errors/`): `AppError` base + `AuthenticationError`, `AuthorizationError`, `ValidationError`, `NotFoundError`, `ConflictError`, `InvitationCodeError`. All carry `code`, `message`, and optional `details`.
- **Constants** (`src/constants/`): card weight multipliers, token TTLs, poll intervals, reveal delay (`3000ms`), connection check endpoint, mutation retry limits.

## Backend Architecture (`apps/backend`)

**Stack:** Next.js 15 App Router, better-sqlite3 (synchronous SQLite), TypeScript.

**Request flow:** `middleware.ts` (CORS + security headers) → `app/api/**/route.ts` (Zod validation + `withAuth`/`withAdmin` HOF) → `lib/services/` (business logic) → `lib/repos/` (prepared statements) → SQLite.

**Key patterns:**

- All queries use prepared statements — no string interpolation.
- Boolean columns use `boolToInt`/`intToBool` bridge helpers (SQLite has no native bool).
- Migrations are timestamp-named TypeScript files in `src/db/migrations/`, applied in order by `pnpm db:migrate` (`src/db/migrate.ts`). **Every schema change requires a migration** — `init.ts` only runs on fresh databases.
- `withAuth` HOF wraps protected routes; `withAdmin` additionally checks `users.is_admin`.
- Every API response uses the `{ ok, data/error, serverTimestamp }` envelope — no exceptions.
- Mutations return the full updated entity so the client can reconcile in one step.
- `GET /api/health` — no auth, no DB access; pure liveness probe.

**Card draw algorithm** (`lib/card-picker/`): weighted random at draw time. Base weight `1.0`, session-card boost `3.0×`, upvote bonus `+0.2` per net vote (cap `+2.0`), downvote penalty `0.5×`, recently-drawn suppression `0.1×`. Excluded cards (flagged/removed or failing session filters) are dropped before weight calculation.

**Admin portal** (`app/admin/`): Mantine UI, protected by a separate admin session (not user JWT), at `/admin`.

## Frontend Architecture (`apps/mobile`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thedinj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
