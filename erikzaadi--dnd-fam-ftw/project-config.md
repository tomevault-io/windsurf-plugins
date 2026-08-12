---
trigger: always_on
description: AI-powered family D&D game. Stack: React 19 + Vite frontend, Node/Express backend, SQLite, OpenAI-compatible AI via the OpenAI SDK.
---

# CLAUDE.md

AI-powered family D&D game. Stack: React 19 + Vite frontend, Node/Express backend, SQLite, OpenAI-compatible AI via the OpenAI SDK.

## Dev setup

```bash
npm run install:all   # install shared, backend, and frontend
npm run dev           # backend :3001 + frontend :5173
```

The repo does not use npm workspaces. The shared package is linked from backend/frontend with `file:../packages/shared`, and `install:all` installs each package explicitly.

All env vars live in the root `.env`.

When debugging a pasted local session URL, fetch the JSON history first. Convert `http://localhost:5173/session/<id>` to `/api/session/<id>/history` on the frontend dev server, or use `http://localhost:3001/session/<id>/history` against the backend directly when it is running.

## Before committing

Do not run token-heavy verification commands yourself unless explicitly asked. This includes `npm test`, `npm run test:*`, `npm run build`, `npm run lint`, `npm run lint:*`, `npx tsc`, and package-specific variants. Ask the user to run the relevant command manually instead.

Manual verification checklist from repo root:

```bash
npm run lint          # all linters: shared → backend → frontend → workflows → bash
npm test              # all tests
npm run tsc           # type-check all packages: shared → backend → frontend
```

Targeted variants:

| Script | What it checks |
|---|---|
| `npm run lint:shared` | ESLint on `packages/shared/src` |
| `npm run lint:backend` | ESLint on `backend/src` |
| `npm run lint:frontend` | ESLint on `frontend/src` |
| `npm run lint:workflows` | actionlint + yamllint + shellcheck on CI workflows |
| `npm run lint:bash` | shellcheck on deploy scripts |
| `npm run tsc:shared` | TypeScript check on `packages/shared` |
| `npm run tsc:backend` | TypeScript check on `backend` |
| `npm run tsc:frontend` | TypeScript check on `frontend` |
| `npm run test:backend` | Backend unit tests |
| `npm run test:frontend` | Frontend unit tests |

Both `lint` and `tsc` follow the same package order: shared → backend → frontend (shared is a dependency of both others). After installing new deps in `packages/shared`, run `npm run install:all` from the root.

## Project layout

```
packages/shared/src/
  types.ts                         # Canonical types shared by backend and frontend - add shared types here

backend/src/
  index.ts                         # Express routes + SSE
  config/env.ts                    # All env var parsing - add new vars here
  services/
    stateService.ts                # SQLite via libsql - all DB access
    authService.ts                 # Google OAuth + JWT
    aiDmService.ts                 # GPT-4o narration
    imageService.ts                # OpenAI-compatible image generation
    gameEngine.ts                  # Dice, damage, turn mechanics
    storySummaryService.ts         # Rolling story compression
  middleware/
    auth.ts                        # Attaches req.namespaceId + req.userEmail
    sessionParam.ts                # Loads namespace-scoped req.session for session id routes
  providers/
    ai/                            # OpenAI-compatible narration + image helpers
    storage/                       # LocalImageStorageProvider + S3ImageStorageProvider
  scripts/
    cli.ts                         # Unified management CLI
    seedSessions.ts                # Seed data (invoked by cli sessions seed)

frontend/src/
  App.tsx                          # Routes + AuthProvider + AuthGuard
  contexts/AuthContext.tsx         # Auth state, enabled/user/logout
  pages/                           # Home, Session, CreateSession, CharacterAssembly, NamespacePicker, RequestInvite, etc.
  components/SiteHeader.tsx        # Banner + back button + logout button
  lib/api.ts                       # apiFetch() and imgSrc() URL helpers - use these everywhere
  stt/                             # Speech-to-text: Web Speech API wrapper, intent parsing, settings hook

terraform/                         # AWS infrastructure
```

## Keeping docs up to date

- **`README.md`** - update if adding a major feature or changing dev/deploy steps
- **`MANAGE.md`** - update if adding/changing CLI commands, deploy scripts, or env vars
- **`GAME_ENGINE_RULES.md`** - update if changing `gameEngine.ts` logic, difficulty, or turn types
- **`DM_PREP.md`** - update if changing DM Prep processing, encounter seed schema, or how seeds trigger
- **`frontend/src/pages/HowToPlay.tsx`** - update when gameplay rules or player-visible flow changes

## Coding conventions

- **No em dashes** - use a hyphen or colon instead
- **All `if` statements must have braces** on a new line (ESLint `curly` rule)
- **Tooltips**: always use `frontend/src/components/Tooltip.tsx`. Reference pattern: `frontend/src/components/game/ActionDock.tsx`. Use `portal` tooltips inside overflow/scroll containers. Never use the native `title` attribute.
- TypeScript strict mode on in all three packages (backend, frontend, packages/shared).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erikzaadi/dnd-fam-ftw](https://github.com/erikzaadi/dnd-fam-ftw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
