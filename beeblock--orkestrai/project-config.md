---
trigger: always_on
description: - Follow the Svelar architecture: route -> controller/page action -> FormRequest/shared schema validation -> DTO -> action/service -> repository -> model/resource -> response.
---

# Svelar App — Agent Guidelines

## Required Flow

- Follow the Svelar architecture: route -> controller/page action -> FormRequest/shared schema validation -> DTO -> action/service -> repository -> model/resource -> response.
- Use Svelar CLI generators before hand-writing artifacts when a generator exists.
- Use Svelar ORM and migrations. Avoid raw SQL unless it is a low-level driver/infrastructure exception.
- Keep one migration per table or focused schema change.
- Use shared schemas for backend validation and frontend forms. Use Superforms where app forms need shared validation.
- Keep validation consistent with `svelar.validation.json`. Use Zod schemas in Zod apps and Valibot schemas in Valibot apps.
- Use policies, permissions, teams, middleware, rate limits, sessions, jobs, events, listeners, observers, cache, storage, search, PDF, and broadcasting through Svelar APIs instead of ad hoc implementations.

## Imports

- Prefer app aliases such as `$lib/modules/...`, `$lib/domain/models/shared/...`, `$lib/database/...`, and `$lib/factories/...`.
- Prefer Svelar subpath imports such as `@beeblock/svelar/orm`, `@beeblock/svelar/routing`, `@beeblock/svelar/forms`, `@beeblock/svelar/validation`, `@beeblock/svelar/auth`, `@beeblock/svelar/queue`, and `@beeblock/svelar/storage`.

## Git And Commits

- Write every commit subject and body in English. Never use Portuguese or Spanish in commit messages.
- Use Conventional Commits with a lowercase type and an imperative, concise subject: `type(optional-scope): summary`.
- Prefer `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`, and `chore`; add a scope when it makes the affected area clearer.
- Write GitHub Release titles and notes in English.
- Keep commits focused on one coherent concern. Do not mix unrelated cleanup or user changes into the same commit.
- Before committing, review the full staged diff and run the verification appropriate to the change. Never commit secrets, runtime databases, generated installers, build output, or local workspace data.

## Frontend

- Use Svelte 5 runes in `.svelte` files: `$props`, `$state`, `$derived`, `$effect`, and `{@render children()}`.
- Do not use Svelte runes in plain `.ts` files.
- Use generated shadcn-svelte components for app UI.
- Mutating browser `fetch` calls must include Svelar's CSRF header. Enhanced forms can use the regular form flow.

## Agent Room Module

- Agent CLIs (claude, codex, kimi, opencode) are accessed only through adapters in `src/lib/modules/agent-room/application/adapters/`. Register new providers via `registerAgentAdapter` in `registry.ts` — never hardcode provider ids outside `domain/types.ts` defaults.
- Agent Room persistence uses Svelar ORM models in `domain/models/` (tables `agent_*`) and repositories in `infrastructure/repositories/`. The legacy better-sqlite3 store (`data/app.sqlite`) was migrated by `npm run migrate:agent-room-data`.
- IDs are UUID v7 (`uuidv7()` from `@beeblock/svelar/support`).
- Legacy data import: `npm run migrate:agent-room-data` (idempotent).
- PTY sessions live in `infrastructure/pty/PtySessionManager.ts` — the singleton MUST stay attached to `globalThis` (the SSR bundle and the type-stripped WS layer load separate module copies; only `globalThis` makes it a true process singleton).
- The PTY WebSocket (`/ws/agent-room/pty`) is served by the vite plugin in dev and by `scripts/orkestrai-server.mjs` in production (HTTP handler + WS in one process; also what Electron spawns). `pty-ws.ts` must stay self-contained (erasable-syntax TS only — Node type stripping runs it).
- The `orkestrai` CLI bridge (ask/list/note/notify/recruit/dismiss/connect/port) lives in `packages/orkestrai-cli` and authenticates per-workspace via `.orkestrai/workspace.json` token written by `BridgeService`. A boot shim (`scripts/install-orkestrai-shim.mjs`, called by both `vite.config.ts` and `scripts/orkestrai-server.mjs`) writes self-contained `orkestrai`/`orkestrai.cmd` launchers into `ORKESTRAI_SHIM_DIR` (`storage/bin` in dev, `<userData>/bin` packaged) which `PtySessionManager` prepends to the PTY `PATH`; packaged shims invoke the Electron executable with `ELECTRON_RUN_AS_NODE=1`, so Windows does not require a separate `node.exe`. Codex's global `~/.codex/config.toml` is repaired on workspace provisioning to use the same absolute runtime + CLI paths (never a bare `.cmd`), and `orkestrai mcp` defers workspace token resolution until a tool call so its global handshake also succeeds outside Orkestrai. The packaged port is dynamic, so the CLI resolves the API URL in this order: `ORKESTRAI_API_URL` env → `~/.orkestrai/runtime.json` (rewritten at every boot) → `workspace.json` apiUrl → default. Agents get their identity via `ORKESTRAI_NODE_ID`/`ORKESTRAI_AGENT_TITLE` env injected at terminal spawn; the CLI uses them as default `--from`/`--agent`.
- Bridge provisioning (skill `.claude/skills/orkestrai/SKILL.md` + `workspace.json`) happens at workspace create AND is repaired lazily in `WorkspaceService.get` (`ensureProvisioned`) — never rely on create-only provisioning for old workspaces. The skill content is re-written when the template changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beeblock/orkestrai](https://github.com/beeblock/orkestrai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
