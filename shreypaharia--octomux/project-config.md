---
trigger: always_on
description: npm package (`octomux`) for orchestrating autonomous Claude Code agents from a web dashboard.
---

# octomux

npm package (`octomux`) for orchestrating autonomous Claude Code agents from a web dashboard.
Single binary: `octomux <command>`. Data stored at `~/.octomux/` in production,
`./data/` in development (`NODE_ENV !== 'production'`).

## Tech Stack

- **Frontend:** Vite + React 19 + Tailwind CSS 4 + shadcn/ui + React Router 7
- **Backend:** Express 5 + better-sqlite3 (WAL mode) + node-pty + ws
- **Terminal:** xterm.js (bidirectional) → node-pty → `tmux attach`
- **Isolation:** git worktrees per task, tmux sessions per task, tmux windows per agent
- **IDs:** nanoid(12)
- **Runtime:** bun (package manager + script runner), tsx (dev server)
- **Binary:** single `octomux` command (`bin/octomux.js`) — `start` launches dashboard, all other subcommands are CLI operations

## Commands

- `bun run dev` — starts Express (7777) + Vite concurrently
- `bun run test` — vitest run (unit + component tests)
- `bun run test:watch` — vitest in watch mode
- `bun run test:e2e` — Playwright E2E tests (auto-starts servers)
- `bun run test:e2e:ui` — Playwright interactive UI mode
- `bun run lint` / `bun run lint:fix` — ESLint 9 flat config
- `bun run format` / `bun run format:check` — Prettier
- `bun run typecheck` — tsc --noEmit
- `bun run build` — Vite build + tsc server

## Architecture

- `server/` — Express backend (API, terminal streaming, task lifecycle, DB)
  - `api.ts` — REST routes mounted on Express app
  - `app.ts` — extracted `createApp()` for testability
  - `task-runner.ts` — worktree + tmux + harness lifecycle (closeTask, deleteTask)
  - `db.ts` — SQLite singleton with `getDb()` / `setDb()` / `initDb()`
  - `logger.ts` — pino root + `childLogger('<module>')` helper
  - `types.ts` — shared types (Task, Agent, TaskStatus, AgentStatus)
  - `harnesses/` — pluggable harness implementations (Claude Code today; Cursor planned).
    Each `Harness` exports `id`, `displayName`, `sessionIdMode`, command builders,
    `installHooks`, `syncAgents`, `resolveFlags`, `validateSettings`. Spec at
    `spec/harness-abstraction.md`; step plan at `plans/2026-05-08-harness-abstraction-step-1.md`.
  - `hook-base-url.ts` — `hookBaseUrl()` returns `http://127.0.0.1:<port>` for harness callbacks.
- `src/` — React SPA (pages, components, lib/api.ts)
- `cli/` — CLI tool for task management (create-task, list-tasks, get-task, close-task)
- `e2e/` — Playwright E2E tests

DB migrations are forward-only. Back up `~/.octomux/octomux.sqlite` (prod) or
`./data/octomux.sqlite` (dev) before upgrading across the harness-abstraction
migration (renames `agents.claude_session_id` → `harness_session_id`, adds
`tasks.harness_id` / `agents.harness_id` / `agents.hook_token`, relaxes
`permission_prompts.session_id` to nullable).

## Logging

- All server-side logs go through `server/logger.ts` (pino). Use
  `const logger = childLogger('<module>');` at the top of each `server/` file and
  emit structured events — `logger.info({ task_id, operation, ... }, 'message')`.
  Never use `console.*` in `server/`.
- Every task/agent lifecycle log line must include `task_id` (and `agent_id`
  where relevant) so grep can reconstruct a timeline:
  `grep '"task_id":"<id>"' ~/.octomux/logs/octomux.log`.
- Output: dev = pretty stdout + rotated JSON at `./data/logs/octomux.log`,
  prod = rotated JSON at `~/.octomux/logs/octomux.log`, test = silent.
- Rotation: daily or 10MB, 7 files kept (pino-roll).
- Default level: `info` in prod, `debug` in dev; override with `LOG_LEVEL`.
- Tests assert log output by piping pino into a buffer via `setLogger(pino({level:'trace'}, stream))`.

## Task Lifecycle

draft → setting_up → running → closed/error
Error at any point → error state with message in `task.error`

Per task: git worktree at `<repo>/.worktrees/<id>`, tmux session `octomux-agent-<id>`,
branch `agents/<id>`. Each agent = tmux window within the session.

- **close** = stop agents + kill tmux session. Preserves worktree and branch (for resume).
- **delete** = kill tmux session + remove worktree + delete branch + delete DB rows. Full cleanup.

## Testing Patterns

- vitest with `NODE_ENV=test` (set in vitest.config.ts)
- Table-driven tests using `it.each()` — prefer over individual test cases
- Shared test harness: `server/test-helpers.ts` (DEFAULTS fixtures, insert/get helpers,
  shell mock assertion helpers via `findExecCall`/`countExecCalls`)
- DB tests use in-memory SQLite via `createTestDb()` → calls `setDb()` for isolation
- task-runner tests mock `child_process` (execFile, spawn) and `fs` (existsSync, mkdirSync, copyFileSync)
- API tests use supertest against `createApp()`
- `CLAUDE_INIT_DELAY` is 0 in test env to avoid 3s sleeps
- `OCTOMUX_AI_TASK_NAMING=1` (or `true`) — optional: on task create with `initial_prompt`, run Claude CLI to polish omitted title/description; off by default so POST `/api/tasks` returns immediately without that subprocess
- E2E: Playwright tests in `e2e/`, config in `playwright.config.ts`
- E2E: `webServer` config auto-starts Express + Vite, reuses running servers in dev
- E2E: helpers in `e2e/helpers.ts` — `createTaskViaAPI`, `waitForStatus`, `deleteAllTasks`, `fillCreateDialog`
- E2E: base-ui Dialog dismisses on Playwright `fill()` — use `click({force:true})` + `pressSequentially` instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShreyPaharia/octomux](https://github.com/ShreyPaharia/octomux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
