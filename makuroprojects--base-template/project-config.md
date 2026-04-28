---
trigger: always_on
description: Default to using Bun instead of Node.js.
---

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

## Server

Elysia.js as the HTTP framework, running on Bun. API routes are in `src/app.ts` (exported as `createApp()`), frontend serving and dev tools are in `src/index.tsx`.

- `src/app.ts` — Elysia app factory with all API routes (auth, admin, logs, presence, hello, health, Google OAuth). Testable via `app.handle(request)`.
- `src/index.tsx` — Server entry. Adds Vite middleware (dev) or static file serving (prod), click-to-source editor integration, audit log rotation, and `.listen()`.
- `src/serve.ts` — Dev entry (`bun --watch src/serve.ts`). Dynamic import workaround for Bun EADDRINUSE race.

## Database

PostgreSQL via Prisma v6. Client generated to `./generated/prisma` (gitignored).

- Schema: `prisma/schema.prisma`
  - `User` (id, name, email, password, role, blocked, timestamps)
  - `Session` (id, token, userId, expiresAt)
  - `AuditLog` (id, userId, action, detail, ip, createdAt)
  - `Ticket` (id, title, description, status, priority, route, reporterId, assigneeId, timestamps, closedAt)
  - `TicketComment` (id, ticketId, authorId, authorTag, body, createdAt)
  - `TicketEvidence` (id, ticketId, kind, url, note, createdAt)
- Enums: `Role` = `USER | QC | ADMIN | SUPER_ADMIN` (default `USER`); `TicketStatus` = `OPEN | IN_PROGRESS | READY_FOR_QC | REOPENED | CLOSED`; `TicketPriority` = `LOW | MEDIUM | HIGH | CRITICAL`
- Client singleton: `src/lib/db.ts` — import `{ prisma }` from here
- Seed: `prisma/seed.ts` — demo users (superadmin, admin, user) with `Bun.password.hash` bcrypt
- Commands: `bun run db:migrate`, `bun run db:seed`, `bun run db:generate`

## Redis

Bun native `Bun.RedisClient` — no external package needed.

- Client singleton: `src/lib/redis.ts` — connects to `REDIS_URL`
- App logs: stored as Redis List (`app:logs`), max 500 entries via `LTRIM`, persists across restart
- App log module: `src/lib/applog.ts` — `appLog(level, message, detail?)`, `getAppLogs(options?)`, `clearAppLogs()`

## Auth

Session-based auth with HttpOnly cookies stored in DB.

- Login: `POST /api/auth/login` — finds user by email, verifies password with `Bun.password.verify`, checks blocked status, creates Session record. Logs to audit trail.
- Google OAuth: `GET /api/auth/google` → Google → `GET /api/auth/callback/google` — upserts user, creates session
- Session: `GET /api/auth/session` — looks up session by cookie token, returns user (including role & blocked) or 401, auto-deletes expired
- Logout: `POST /api/auth/logout` — deletes session from DB, clears cookie
- Blocked users: login returns 403, existing sessions are invalidated on block, frontend redirects to `/blocked`

## Admin API (SUPER_ADMIN only)

- `GET /api/admin/users` — list all users with role, blocked status, createdAt
- `PUT /api/admin/users/:id/role` — change role to USER or ADMIN (cannot change self or to SUPER_ADMIN)
- `PUT /api/admin/users/:id/block` — block/unblock user (deletes all sessions on block)
- `GET /api/admin/presence` — list online user IDs
- `GET /api/admin/logs/app` — app logs from Redis (filter: level, limit, afterId)
- `GET /api/admin/logs/audit` — audit logs from DB (filter: userId, action, limit)
- `DELETE /api/admin/logs/app` — clear all app logs from Redis
- `DELETE /api/admin/logs/audit` — clear all audit logs from DB
- `GET /api/admin/routes` — all routes metadata (method, path, auth level, category, description) with summary stats
- `GET /api/admin/project-structure` — scans `src/`, `prisma/`, `tests/` — returns files with line counts, exports, imports, categories + directory tree
- `GET /api/admin/env-map` — environment variables with set/unset status, required/optional, default values, consuming files
- `GET /api/admin/test-coverage` — source files + test files mapping, coverage status (covered/partial/uncovered)
- `GET /api/admin/dependencies` — NPM packages from package.json with version, type (runtime/dev), category, importing files
- `GET /api/admin/migrations` — Prisma migration timeline with parsed SQL changes and date info
- `GET /api/admin/sessions` — all active sessions with user info, online status, expiry, role breakdown

## Tickets API

Role-gated ticket tracking. Status machine: `OPEN → IN_PROGRESS → READY_FOR_QC → CLOSED` with `REOPENED` branch. Allowed-transition helper enforces valid moves.

- `GET /api/tickets` — list (QC users see only QC-scope tickets)
- `POST /api/tickets` — create (any authed user reports)
- `GET /api/tickets/:id` — detail with comments + evidence
- `PATCH /api/tickets/:id` — update status/priority/assignee (role-gated)
- `POST /api/tickets/:id/comments` — add comment
- `POST /api/tickets/:id/evidence` — attach evidence (url + kind)

Frontend: `src/frontend/components/TicketsPanel.tsx` — shared between `/dev` and `/dashboard`. Filtered to QC scope when user is QC.

## MCP Server


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makuroprojects) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
