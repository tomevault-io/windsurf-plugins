---
trigger: always_on
description: Orchestrator — a local-first web app that runs many Claude Code sessions in parallel across multiple projects from one screen. Each **project** carries reusable context + a working directory; each **task** is its own Claude Code session in its own git worktree, driven by `@anthropic-ai/claude-agent-sdk` against the user's local Claude login (no API key). (A hosted version, getoperator.dev, lives in a separate private repo that overlays this one — see "Repo split" below.)
---

# CLAUDE.md

Orchestrator — a local-first web app that runs many Claude Code sessions in parallel across multiple projects from one screen. Each **project** carries reusable context + a working directory; each **task** is its own Claude Code session in its own git worktree, driven by `@anthropic-ai/claude-agent-sdk` against the user's local Claude login (no API key). (A hosted version, getoperator.dev, lives in a separate private repo that overlays this one — see "Repo split" below.)

## Commands

- `npm run dev` — app (:3000, `server.js`) + pty sidecar (:3001, `pty-server.js`) via concurrently. `npm run dev:next` / `npm run pty` run them separately.
- `npm run build` (turbopack) then `npm start` for production.
- `npm test` — vitest, serial on purpose (tests spawn many real git subprocesses). Single file: `npx vitest run tests/merge.test.ts`.
- `npm run test:e2e` — Playwright end-to-end suite: builds, then boots the real prod server against a hermetic temp instance with the deterministic mock agent (`lib/agents/mock/`, registered only when `ORCH_E2E_MOCK_AGENT=1`) and drives onboarding → project → task → turn → diff → merge through the UI. `npm run preflight` = unit + e2e, the pre-push gate. See `e2e/README.md` (mock-turn directives, selector conventions, staleness gotcha: the server runs the **built** bundle).
- No lint script; TypeScript is strict, path alias `@/*` → repo root (mirrored in `vitest.config.ts`).

## Architecture

Three processes/entrypoints, one origin:

- **`server.js`** — custom Next.js server (plain Node, CommonJS). Fronts Next on one port, proxies `/pty` WebSocket upgrades to the sidecar, forwards dev HMR upgrades to Next, enforces origin auth on WebSocket upgrades (middleware never sees upgrades — this file is the auth boundary for the terminal), and dispatches public service hostnames (`<slug>--<appHost>`) through `lib/service-router.mjs`.
- **`pty-server.js`** — node-pty sidecar, bound to `127.0.0.1` only; never exposed directly.
- **Next app** — UI in `app/`, REST under `app/api/`, server logic in `lib/`.

### The turn lifecycle (core flow)

`POST /api/tasks/[id]/messages` doesn't run the turn — it calls `startTurn()` in **`lib/runner.ts`** and returns. The turn runs detached, owned by the server process: every event is persisted to SQLite and fanned out via **`lib/events.ts`** (in-process pub/sub keyed by task id, plus a wildcard channel — `subscribeGlobal()` — that sees every task's events). `GET` on the same route is the SSE watch stream: a `snapshot` of the persisted transcript, then a live tail — reconnect-safe, any number of viewers, zero viewers fine. Stopping is only explicit (`lib/abort.ts`). If a turn is already running, POST parks the message in `pending_messages` to run next.

Only the SELECTED task has a transcript stream open. Everything else stays live through `GET /api/events` — one always-open EventSource per tab (`app/orchestrator/useGlobalEvents.ts`) broadcasting coarse lifecycle events for every task across every project (turn started / awaiting input / answered / suggestion created / turn ended). Each payload re-reads the task row at publish time — the runner persists BEFORE it publishes, so the snapshot is authoritative (pinned by `tests/agentDriver.test.ts`); it also carries the project's fresh awaiting count. That's what updates spinners, project badges, and the "N need you" pill for unselected tasks — there is no task-list polling.

**`lib/agents/`** is the agent-driver seam: the app talks to coding agents only through the `AgentDriver` interface (`types.ts` — normalized `StreamEvent` turn contract, one-shot summarize/draft/recap helpers, capability descriptor, login/verify auth surface), resolved via `getDriver(task.agent)` in `registry.ts` (`tasks.agent`, defaulted from `projects.default_agent`; unknown ids fall back to Claude). `shared.ts` holds the agent-agnostic normalizers (project-context/conflict prompts, tool-call → title/peek/diff, the event queue). `GET /api/agents` exposes each driver's capabilities to the client. Session/thread ids are opaque per driver (`sessions.claude_session_id` stores any driver's id).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iishyfishyy/operator-oss](https://github.com/iishyfishyy/operator-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
