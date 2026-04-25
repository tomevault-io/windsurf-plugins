---
trigger: always_on
description: 3D Pac-Man web game rendered as an office building using React Three Fiber. Each level is a floor (10 total), connected by elevator mechanic. Frontend SPA with a FastAPI leaderboard backend.
---

# 3D Pac-Man Web Game

## Project Overview
3D Pac-Man web game rendered as an office building using React Three Fiber. Each level is a floor (10 total), connected by elevator mechanic. Frontend SPA with a FastAPI leaderboard backend.

## Tech Stack
- **Build**: Vite 8.x
- **UI**: React 19.2 + TypeScript 5.9
- **3D**: React Three Fiber v9 + Drei v10
- **State**: Zustand v5
- **Serve**: nginx:alpine (Docker)
- **Deploy**: Docker + Traefik + GitHub Actions → pacman.enricd.com

## Commands

### Frontend
- `npm run dev` — Start dev server with HMR
- `npm run build` — TypeScript check + Vite build
- `npm run lint` — ESLint
- `npm run preview` — Preview production build

### Backend (run from `backend/`)
- `uv run fastapi dev app/main.py` — Start API dev server with hot reload
- `uv run pytest` — Run all backend tests (65 tests)
- `uv run pytest -v` — Run tests with verbose output
- `uv run pytest tests/test_submit_score.py` — Run a specific test file
- `uv lock` — Lock dependencies after changing `pyproject.toml`
- `uv sync --dev` — Install all dependencies including dev (pytest)

## Skill Files (`.claude/skills/`)

**Always check the relevant skill file before writing code that uses a library.** Skill files contain verified, up-to-date API references and patterns for each tool in the stack.

| Skill File | Covers |
|------------|--------|
| `react-three-fiber.md` | R3F v9 Canvas, useFrame, useThree, InstancedMesh, events + Drei v10 helpers |
| `zustand.md` | Zustand v5 create, selectors, getState, middleware, R3F integration pattern |
| `three-js.md` | Three.js 0.183 geometries, materials, lights, cameras, InstancedMesh, math |
| `vite-react-ts.md` | Vite 8 config/HMR/env + React 19 hooks/patterns + TypeScript 5.9 rules |
| `deployment.md` | Traefik labels, compose patterns, GitHub Actions CI/CD, nginx SPA config |
| `fastapi-backend.md` | FastAPI, SQLModel, SlowAPI, pydantic-settings, uv — verified API patterns |

### Rules for Using Skill Files
1. **Read the relevant skill file** before writing or modifying code that uses that library. Do not rely solely on training knowledge — the skill file has the verified, version-specific API.
2. **If a skill file doesn't cover what you need**, use `npx ctx7@latest` to fetch the latest docs (the lookup command is at the top of each skill file), then **update the skill file** with the new information.
3. **Keep skill files current**: When upgrading a dependency or discovering a new pattern, update the relevant skill file so future conversations benefit.
4. **Use the find-docs skill** (`npx ctx7@latest docs <libraryId> "<query>"`) when you encounter an API you're unsure about. Never guess at API signatures — look them up.

## Architecture Rules

1. **Pure game logic in `systems/`** — No React or Three.js imports. Just functions that take state and return new state.
2. **Single Zustand store** (`stores/gameStore.ts`) — No prop drilling, no React context for game state.
3. **InstancedMesh for repeated geometry** — Walls (`Wall.tsx`) and pellets (`Pellet.tsx`) use instancing.
4. **Constants in `utils/constants.ts`** — No magic numbers in components or systems.
5. **Grid-based movement** — All game logic operates on integer grid positions. 3D is rendering only.
6. **`gridToWorld()` for coordinate conversion** — Grid (row, col) → world (x, z). Y is up.
7. **Frame-rate independent** — All movement uses `delta` from `useFrame`.

## Project Structure
- `src/types/` — TypeScript type definitions
- `src/stores/` — Zustand store
- `src/hooks/` — React hooks (useKeyboard, useGameLoop)
- `src/systems/` — Pure game logic (movement, collision, ghostAI, mazeGenerator)
- `src/components/` — R3F 3D components (maze, characters, items, mechanics, camera)
- `src/ui/` — HTML overlay components (HUD, screens)
- `src/utils/` — Constants and helpers
- `src/api/` — Frontend API client (leaderboard)
- `.claude/skills/` — Up-to-date API reference docs for each library
- `backend/app/` — FastAPI leaderboard API (config, models, routes, database)
- `backend/tests/` — Backend pytest test suite

## Testing

### Backend Tests (`backend/tests/`)
Run: `cd backend && uv run pytest -v`

Uses **in-memory SQLite** (via `StaticPool`) and FastAPI's `dependency_overrides` for isolated tests. Rate limiting is disabled in most fixtures and tested separately.

| File | Covers | Count |
|------|--------|-------|
| `test_health.py` | Health endpoint returns 200 | 1 |
| `test_submit_score.py` | Valid submissions, all validation rules (username, score, level, duration), missing/invalid fields, type errors | 30 |
| `test_leaderboard.py` | Empty/populated leaderboard, sort order, tiebreaker, limit bounds, response shape, IP exclusion | 13 |
| `test_models.py` | Pydantic model validation independent of API (boundaries, unicode, emoji, stripping) | 13 |
| `test_rate_limiting.py` | Rate limit allows 5/hour, blocks 6th, doesn't affect GET endpoints | 4 |

**Rules:**
1. **Always run tests** after modifying backend code: `cd backend && uv run pytest`
2. **Add tests** for any new endpoint or validation rule
3. **Each test file** gets its own `conftest.py` fixture where needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enricd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
