---
trigger: always_on
description: Dockerised sandbox: non-technical users describe CRM changes in chat → agent team ships them in git worktrees.
---

# Atomic-CRM Builder

Dockerised sandbox: non-technical users describe CRM changes in chat → agent team ships them in git worktrees.

## Runtime

```
supervisord (pid 1)
  ├─ crm-frontend   :5173  (Vite, /app/src)
  └─ chat-service   :8080  (WebSocket + persistent PTY orchestrator)
```

Two compose profiles: `demo` (FakeRest) and `full` (Supabase, needs Docker socket).

`entrypoint.sh`: syncs `claudeConfig/.claude/` → `/home/developer/.claude`, applies App.tsx variant, overwrites `/app/.claude/settings.json` with `{"hooks":{}}` (prevents upstream format-file.sh fight with our hooks).

Single `crm-app` volume for `/app` — keeps `node_modules` and `worktrees/` on the same device so `cp -al` hard-links node_modules into each worktree (zero disk cost).

## Chat-service (`chat-service/`)

Node.js server (:8080) — WebSocket + REST. Entry: `server.js`. Split: `lib/server/` + `lib/stats/`.

Architecture reference: [docs/chat-service-architecture.md](docs/chat-service-architecture.md)

Key modules:
- `server/turn-helpers.js` — turn helpers: assistant-message extraction, `FULL_SETUP` intent rewrite, resume planning (fresh vs `--resume`), user-facing error text (formerly `claude-spawn.js`; the PTY model removed headless spawning)
- `server/turn.js` — streams stdout, writes `log.jsonl`, recovery decision, snapshots transcripts — see [docs/turn.md](docs/turn.md)
- `server/session-store.js` — chat UUID, `meta.json`, `TASK-*.json` detection
- `server/subagent-tail.js` — polls subagents/ every 2500ms → WS broadcast
- `lib/stats/` (8 modules) — read-only aggregation, `GET /api/stats` — see [docs/stats.md](docs/stats.md)
- `server/deploy-routes.js` — SSE `/api/deploy/events`, 6-phase pipeline (vite build → supabase link → db push → functions → secrets → wrangler), independent of chat WS — see [docs/deploy.md](docs/deploy.md)

Tests: `cd chat-service && npm test` — uses glob `'test/**/*.test.js'` (directory form broken on Node 25).

## Agent team (`claudeConfig/.claude/agents/`)

| Agent | Model | Role |
|---|---|---|
| chat-orchestrator | sonnet | User-facing, routes, narrates. SIMPLE flow dispatches simple-developer + merger directly (no team). |
| planner | sonnet | Decomposes → tickets JSON with waves + file hints. |
| developer | opus | Implements + commits in worktree. Also writes ADRs in `adr/` when the change introduces a structural decision. Never writes SQL migrations — deploy-time only. |
| simple-developer | sonnet | 1-file cosmetic OR 1 single-field change on an existing entity (schema + view + type + form + show) OR 1 list filter reusing existing components (no new custom React component). No team, no review, never writes ADRs — SubagentStop hooks validate. POST-DEV runs if a migration was written. |
| quality-reviewer | sonnet | Semantic code + security review only. Never re-runs validation. |
| test-validator | sonnet | Integration wiring + e2e presence. (Deliberately bumped from haiku in #17.) |
| merger | haiku | `git merge --no-ff` only. **Never `git add`/`git commit`**. |
| documentator | sonnet | Mode 1 — captures rules/skills to `~/.claude/local/` on explicit user request. Mode 2 — dispatched by the orchestrator (via `Agent`, run_in_background) at POST-DEV once the user validates the result; appends business knowledge to `/app/MEMORY.md` from the diff vs `origin/main`. Both modes run via the Agent tool (no `claude -p`); confined by `restrict-documentator-{write,bash}` (which trigger on `agent_type=documentator` or the legacy `DOCUMENTATOR_RUN=1`). |

Dispatch layout (no team): the orchestrator dispatches each wave's agents directly via the `Agent` tool — one developer per ticket, then 2 reviewers per ticket, then the merger. Merges run one at a time into the `_session` worktree, so there is no `.git/index.lock` contention.

### Hooks (`claudeConfig/.claude/settings.json`)

No-team flow: the orchestrator dispatches every agent via the `Agent` tool (no `TeamCreate`/`SendMessage`), so the worktree + dispatch guards live on `PreToolUse / Agent`.

- `PreToolUse / Bash` → silent-mode-check, circuit-breaker, block-bash-file-write, block-bash-validation, block-orchestrator-merge, restrict-documentator-bash
- `PreToolUse / Write|Edit` → restrict-documentator-write, block-migration-writes
- `PreToolUse / Agent` → block-merger-without-review, **block-promote-unmerged** (refuses `MODE: promote` while any `<id>/*` task branch has commits not on `session/<id>`), enforce-dev-dispatch, **setup-worktree** (creates the worktree from the dispatch prompt's `WORKTREE_PATH`/`BRANCH_NAME`/`TASK_ID` BEFORE the agent starts — replaces the old `SubagentStart` trigger, which couldn't see the TASK id in no-team mode)
- `SubagentStop / merger` → cleanup-worktree
- `SubagentStop / quality-reviewer|test-validator` → record-review-verdict
- `SubagentStop / simple-developer|developer` → typecheck, prettier, unit-app, unit-functions, e2e


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmelab/crm-builder](https://github.com/marmelab/crm-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
