---
trigger: always_on
description: Self-hosted workout tracker. Go backend, React/TypeScript frontend, SQLite, Docker.
---

# Lyftr — project context

Self-hosted workout tracker. Go backend, React/TypeScript frontend, SQLite, Docker.
GitHub: https://github.com/Cawlumm/lyftr

Written for everyone who works on this repo, human or agent. It is checked in, so it is
also what the automated PR review reads; `REVIEW.md` says how findings against these rules
should be weighed, and deliberately does not restate them.

Machine-specific setup — your own paths, ports, editor, hardware limits — does not belong
here. Put that in your personal `~/.claude/CLAUDE.md`, which is not shared.

Every rule below was paid for by a bug. Where one names a file, that file is the single
place the rule is enforced; adding a second is how the rule stops holding.

---

## Stack

| Layer | Tech |
|-------|------|
| Backend | Go 1.26, Gin, SQLite (`_foreign_keys=on`) |
| Frontend | React 18, TypeScript, Tailwind CSS, Vite, Recharts |
| Auth | JWT + refresh tokens (middleware/auth.go) |
| Deployment | Docker Compose, nginx reverse proxy |

---

## Project Structure

```
backend/
  main.go               — entry point, wires DB + routes + seed
  config/config.go      — env vars (JWT_SECRET, CORS_ORIGIN, PORT)
  controllers/          — one file per resource
  db/sqlite.go          — opens SQLite, enables WAL + foreign keys
  db/migrations.go      — schema migrations (run on startup)
  middleware/auth.go    — JWT validation middleware
  models/models.go      — all structs + request types
  routes/routes.go      — all routes, protected vs public
  stores/exercise.go    — exercise store, incl. free-exercise-db seeding
  seed/users.go         — demo user seeding (see also seed/demo_data.go)
  utils/response.go     — utils.OK / utils.BadRequest helpers

web/src/
  pages/                — one file per page/route
  services/api.ts       — all API calls (axios), typed
  (types come from @lyftr/shared, re-exported via services/api.ts)
  App.tsx               — React Router routes
```

---

## Key Conventions

**Backend**
- All handlers use `utils.OK(c, data)` and `utils.BadRequest(c, msg)`
- Routes: public under `/api/v1/auth/`, protected under `/api/v1/` (JWT required)
- Admin routes: `protected.GET/POST("admin/...")` — requires auth, no separate role check yet
- Weight stored in **lbs** internally (backend stores raw numbers; the unit is a frontend convention). Frontend converts via `lbsToDisplay`/`displayToLbs` in `web/src/stores/settings.ts` based on `user_settings.weight_unit` — never store display values directly
- `exercise_id` FK references `exercises(id)` — **no ON DELETE CASCADE** — never wipe exercises table while workout/program data exists
- New migrations go in `db/migrations.go` — append only, never modify existing

**Frontend**
- API calls go through `web/src/services/api.ts` — add new endpoints there, not inline
- Always null-guard arrays before `.reduce`/`.forEach`: `(w.exercises ?? []).forEach(...)`
- Duration stored as **seconds** in DB — display as `Math.round(duration / 60)` min
- Light/dark theme via `localStorage.setItem('theme', 'light|dark')` + `document.documentElement.classList`
- Mobile-first: 90% of users on mobile, desktop is secondary
- New pages: create in `web/src/pages/`, add route in `App.tsx`, add API methods in `api.ts`

---

## Time & Calendar Days — read before touching any date

**The rule: a row records its own day. Nothing re-derives one from whoever is reading.**

An instant does not contain a day — a day is an instant *plus a place*. So every
day-scoped row stores both. This is the model Fitbit, Garmin, Strava and Android
Health Connect all use, and the reason travelling no longer rewrites history.

| Entity | Instant | Day attribution | Peer precedent |
|---|---|---|---|
| `food_logs`, `weight_logs` | `logged_at` (UTC) | `logged_on` — stored `YYYY-MM-DD` | Fitbit `logDate` |
| `workouts` | `started_at` (UTC) | `tz_offset_minutes` — offset at start | Strava `utc_offset`, Garmin `startTimeOffsetInSeconds` |

Diary entries store the day because the user *picks* it. Workouts store an offset
because the day *follows* from where the moment happened. Fitbit splits them the
same way.

**`user_settings.timezone` is not dead and is not the source of day attribution.**
It answers only questions about *now*, which no stored row can answer:
- "What is today?" for a read with no `date` param
- "30 days back from *your* today" — `/food/history`, `/weight/stats` send no dates
- The day/offset for a write from a client too old to send one (back-compat fallback)

### Single points of truth — do not add a fifth way to get a day

- **Backend**: `backend/controllers/timezone.go`. `resolveDay` (writes),
  `resolveQueryDay` (reads), `daysAgoDay` (windows), `tzOffsetMinutes` (workouts).
  Every handler goes through one of these. No inline `time.Now().In(loc)` elsewhere.
- **Client**: `packages/shared/src/utils/dateUtils.ts` — **one copy, imported by both
  apps** from `@lyftr/shared`. `entryDay(e)` for diary rows, `workoutDay(w)` for
  workouts — never `new Date(x).getDate()` or `.toISOString().slice(0,10)` in a page.
  Writes go through `withLoggedOn` / `utcOffsetMinutes` in the API layer, so no screen

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cawlumm/lyftr](https://github.com/Cawlumm/lyftr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
