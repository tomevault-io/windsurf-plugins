---
trigger: always_on
description: > Read this file at the start of every task. Rarely modified — only update when project structure or conventions change.
---

# CLAUDE.md
> Read this file at the start of every task. Rarely modified — only update when project structure or conventions change.

## Universal Rules
- Think from first principles. Don't assume the user knows exactly what they want or the best way to get it. Start from the original requirement, question the approach, and suggest a better path if one exists
- Think step by step. Investigate before coding — read relevant code, trace the full flow, print findings before proposing a fix
- When a task is complex, break it into sub-tasks and spawn sub-agents to work in parallel
- Never guess. If unsure, read the code, check logs, or run a test first
- Every task must produce a visual verification artifact (screenshot, plot, diff, rendered output)
- If the goal or motivation is unclear, stop and discuss before writing code. If the goal is clear but the path isn't optimal, say so and suggest the better approach
- For "does X cause Y in the UI" questions, trace the full chain: backend write → event emit → API caller → component state → render condition. Grepping the backend alone is insufficient — feedback often comes from local setState or per-entity refetch, not WS
- Match assertion strength to evidence. If you only verified part of a claim, say so explicitly — don't write "❌ doesn't trigger" / "0–5s delay" when you only checked one of multiple paths. Downgrade language ("backend doesn't, frontend not checked") instead of presenting a partial check as complete

## Do NOT
- Do not refactor or rename files unless the task explicitly requires it
- Do not delete or modify tests unless asked
- Do not change dependencies/package versions without explicit approval
- Do not modify CLAUDE.md
- Do not write to memory files (.claude/memory/, MEMORY.md) — only the orchestrator manages persistent memory

## Output Rules
- Keep responses concise — no long explanations unless asked
- For large outputs (logs, data), write to a file instead of printing to stdout
- Truncate error logs to the relevant section, don't paste entire stack traces

## Git Conventions
- Commit message format: `[scope] brief description` (e.g. `[frontend] fix image zoom gesture`)
- Commit frequently — small atomic commits, not one giant commit at the end
- Commit to master directly when appropriate

## Concurrency Rules
- Check which files other agents are currently modifying before editing shared files
- Prefer creating new files over modifying existing shared ones when possible

## Code Style
- Follow existing patterns in the codebase — don't introduce new conventions
- Match the indentation, naming, and structure of surrounding code

## Project: cc-orchestrator (Xylocopa, formerly AgentHive)
- Tech Stack: Python 3.11+ (FastAPI), React (Vite), SQLite
- Top Dirs: certs/, frontend/, orchestrator/, project-configs/, projects/
- Config: .env
- Entry: orchestrator/main.py
- Tests: test_multi_question.py
- Build (frontend): `cd frontend && npx vite build && pm2 reload xylocopa-frontend` — required after a frontend-only edit (prod-build mode, no HMR). Skip the manual build if you're about to restart anyway: both `./run.sh restart` and `/api/system/restart` auto-detect stale dist/ and rebuild first
- Test (frontend): `cd frontend && npx vitest run`
- Verify backend: `cd orchestrator && ../.venv/bin/python -c "from models import *; print('OK')"` (use the project venv — system python lacks the deps)
- Restart: `./run.sh` or POST `/api/system/restart` (both auto-rebuild stale frontend before restart)
- Logs: `logs/server.log`, `logs/orchestrator.log`

## Xylocopa context
- This project is managed by xylocopa. The orchestrator MCP server is auto-registered via `.mcp.json`.
- Available tools: `project_*`, `task_*`, `session_*`, `agent_*`, `system_health` — list/get/read/create/update/dispatch/scaffold only, no destructive verbs.
- Full reference: xylocopa repo `docs/agent-mcp-tools.md`.

## Project-Specific Rules
See README.md for detailed project documentation.
- Worktree sessions: always use `_resolve_session_jsonl()`, never bare `session_source_dir()`
- tmux pane matching: `xy-{agent_id[:8]}` (legacy `ah-{agent_id[:8]}` still recognized) session name is authoritative
- CWD matching: use `startswith(proj + "/")` not `==` (worktree subdirs)
- SQLAlchemy: `metadata` is reserved — use alt attr name with explicit column
- When fixing a helper, grep ALL call sites — don't assume you found them all
- Queued messages use stop-hook dispatch: PENDING in DB → stop hook fires → send via tmux → UserPromptSubmit confirms delivery

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyao97/xylocopa](https://github.com/jyao97/xylocopa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
