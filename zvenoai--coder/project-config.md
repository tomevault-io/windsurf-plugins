---
trigger: always_on
description: > **CRITICAL: Every PR review comment — write a test first, then fix. Never fix blindly. See "Bug Fix Process" section.**
---

# ZvenoAI Coder

> **CRITICAL: Every PR review comment — write a test first, then fix. Never fix blindly. See "Bug Fix Process" section.**
>
> **CRITICAL: NEVER push to main or merge a PR until ALL quality checks pass (`task quality` — lint, format, typecheck, tests). Zero failures allowed. Pre-existing failures must be fixed first.**

Python-based async orchestrator that polls Yandex Tracker for tasks tagged `ai-task` and dispatches Claude Agent SDK agents to execute them. Includes a real-time web dashboard for monitoring agent output and task status.

## Stack
- Python 3.12+, claude-agent-sdk, FastAPI + uvicorn, requests, PyYAML
- React 19 + Vite + TypeScript + TailwindCSS, xterm.js
- pytest + pytest-asyncio

## Architecture
- **Agent SDK** — in-process execution via `ClaudeSDKClient`
- **Orchestrator Agent** — `OrchestratorAgent` handles worker result decisions (track PR, complete, fail, epic child events)
- **Agent-driven completion** — agent decides when task is done; PR tracking is informational
- **In-process MCP tools** — Tracker tools scoped per-issue (no external MCP processes)
- **Git worktrees** — per-task workspace isolation
- **Error recovery** — typed error classification + exponential backoff retry
- **Async concurrency** — `asyncio` with semaphore-controlled parallel agents
- **Epic coordination** — supervisor-driven: auto-discovery of children (with auto-decomposition), dependency graph via MCP tools (`awaiting_plan` → `executing`), lifecycle events, `epic_reset_child`
- **Workpad** — persistent structured comment on Tracker issue; idempotent via hidden HTML marker; agent updates progress via MCP tools
- **Tracker status reconciliation** — periodic check for externally closed/cancelled tasks; phase-aware cleanup; `removed` flag prevents stale reference races
- **Multi-turn continuation** — retries agent up to `MAX_CONTINUATION_TURNS` (3) when it completes without PR and task is still open; `tracker_mark_complete` for explicit no-PR completion; cost cap guard (`MAX_CONTINUATION_COST`)
- **Merge conflict retry** — SHA-gated retry (up to `MERGE_CONFLICT_MAX_RETRIES` = 2); resets on resolution
- **Heartbeat monitor** — periodic health checks (every 5 min) detecting stuck agents, long-running tasks, stale reviews; cooldown-based deduplication
- **PR auto-merge** — opt-in when CI green + reviews approved + no conflicts; GitHub GraphQL `enablePullRequestAutoMerge` with REST fallback
- **Pre-merge code review** — one-shot Sonnet sub-agent reviews PR diff; fail-close by default (`PRE_MERGE_REVIEW_FAIL_OPEN`); posts REQUEST_CHANGES on reject; auto-resets on new commits for fresh review cycles
- **Human gate** — blocks auto-merge for large diffs (`HUMAN_GATE_MAX_DIFF_LINES`) or sensitive paths (`HUMAN_GATE_SENSITIVE_PATHS`)
- **Post-merge verification** — watches CI + K8s rollout, spawns verification sub-agent on dev; on fail auto-creates hotfix Bug task; configurable via `POST_MERGE_VERIFICATION_ENABLED`
- **Environment config** — SQLite key-value store for per-environment connection details; supervisor writes via `env_set`, workers read via `env_get`
- **Event bus** — async pub/sub for real-time streaming to web dashboard
- **Web dashboard** — FastAPI REST + WebSocket, React frontend with xterm.js
- **Supervisor chat** — interactive + autonomous streaming with `bypassPermissions`; `auto_send()` for epic planning
- **Supervisor memory** — SQLite + FTS5 hybrid search (BM25 0.3 / vector 0.7) over markdown files in `data/memory/`; Gemini embeddings; auto-refresh on content change
- **Inter-agent communication** — centralized `AgentMailbox` with interrupt-based message delivery; message types: REQUEST, RESPONSE, NOTIFICATION, ARTIFACT; delivery statuses: DELIVERED, QUEUED, OVERFLOW_DROPPED (MAX_INBOX_SIZE=50); 5 MCP tools per agent; supervisor has read-only access
- **Task dependency management** — auto-defers tasks with unresolved deps (Tracker links + LLM text extraction via Haiku); supervisor can override; fail-open on errors
- **K8s diagnostics** — optional pod log/status inspection; feature-gated via `K8S_LOGS_ENABLED`
- **Persistent stats** — SQLite-backed via EventBus subscriber
- **Auto-compaction** — Haiku summarization when approaching token limit, session recreation
- **Session resumption** — persists `session_id` in SQLite; resumes with `fork_session=True`; fresh fallback with context prompt on failure
- **Dead session recovery** — auto-recreates failed sessions with context preserved; publishes `SESSION_RECREATED` event

## Python Style Guide (based on Google Python Style Guide)

Follows [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) with additions below.

### Imports
**Ordering** (blank-line separated, lexicographic within group):
1. `from __future__` 2. stdlib 3. third-party 4. local

- `import x` for packages/modules only — `from x import y` for submodules
- Never use relative imports — always full package path
- `typing`, `collections.abc`, `typing_extensions` symbols may be imported directly

### Naming
| Type | Convention | Example |
|------|-----------|---------|
| Modules | `lower_with_under` | `agent_runner.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zvenoai/coder](https://github.com/zvenoai/coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
