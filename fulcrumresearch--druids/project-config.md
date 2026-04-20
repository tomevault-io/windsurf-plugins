---
trigger: always_on
description: This file provides guidance for coding agents (e.g., Claude Code, Codex, Gemini CLI) working on Druids, a multi-agent orchestration system.
---

# Agent Guidance

This file provides guidance for coding agents (e.g., Claude Code, Codex, Gemini CLI) working on Druids, a multi-agent orchestration system.


## Commands

All commands are relative to the repo root. Server configuration lives in `server/.env` (see `server/druids_server/config.py` for all settings).

### Developing

```bash
cd server && uv run druids-server          # start the server (port from .env, default 8000)
cd server && uv run pytest                 # run server tests
cd server && uv run pytest --slow          # include slow tests
cd server && uv run pytest tests/api/      # run a specific test directory
cd client && uv run pytest                 # run client tests
ruff check --fix                           # lint and autofix
ruff format                                # format code
pre-commit run --all-files                 # run all pre-commit hooks (ruff check + format)
```

Before starting the server for the first time (or after model changes), run `cd server && uv run alembic upgrade head` to apply database migrations. On startup the server builds the client wheel (`client/dist/*.whl`). Server logs append to `server/logs/druids.log`.

### Database Migrations

Alembic is the sole owner of database schema. Run migrations before starting the server:

```bash
cd server && uv run alembic upgrade head                        # apply all migrations
cd server && uv run alembic revision --autogenerate -m "desc"   # create a new migration
cd server && uv run alembic downgrade -1                        # roll back one migration
cd server && uv run alembic history                             # list migration history
```

### Running Programs

Programs are Python files in `.druids/`. The server must be running.

```bash
druids exec build spec="implement feature X"               # run a program
druids exec build -b my-branch spec="..."                  # run on a specific git branch
druids execution ls                                        # list running executions
druids execution ls --all                                  # include stopped executions
druids execution status SLUG                               # check execution status
druids execution activity SLUG                             # show recent agent activity
druids execution stop SLUG                                 # stop an execution
druids execution send SLUG "try a different approach"      # send a message to the builder agent
druids execution send SLUG "check the logs" -a monitor     # send to a specific agent
druids execution ssh SLUG                                  # open a shell on execution VM
druids execution ssh SLUG -a agent-name                    # shell into a specific agent's VM
druids execution connect SLUG                              # resume the agent's coding session
druids execution connect SLUG -a agent-name                # resume a specific agent's session
```

The CLI reads `~/.druids/config.json` for `base_url` and auth token.

### Execution Traces

Traces are JSONL at `~/.druids/executions/{user_id}/{slug}.jsonl`. Each line is a JSON object with `ts`, `type`, `agent`, and event-specific fields.

```bash
cat ~/.druids/executions/*/SLUG.jsonl                                       # read full trace
tail -20 ~/.druids/executions/*/SLUG.jsonl                                  # last 20 events
cat ~/.druids/executions/*/SLUG.jsonl | jq -r 'select(.type == "error")'    # filter errors
```

Event types: `execution_started`, `connected`, `disconnected`, `prompt`, `response_chunk`, `tool_use`, `tool_result`, `execution_stopped`, `error`, `topology`, `client_event`.

### Frontend

The dashboard is served at the server's root URL when `frontend/dist/` exists.

```bash
cd frontend && npm install                     # install dependencies
cd frontend && npm run dev -- --host 0.0.0.0   # start Vite dev server (bind all interfaces)
cd frontend && npm run build                   # production build (creates dist/)
```

### Devbox Management

```bash
druids devbox create --repo owner/repo                # provision a devbox sandbox
druids devbox snapshot --name owner/repo              # snapshot and stop the devbox
druids devbox ls                                      # list all devboxes
druids devbox secret set -d devbox-name KEY VALUE     # set a secret on a devbox
druids devbox secret ls -d devbox-name                # list secrets
```

### Docker Sandbox Backend

When `DRUIDS_SANDBOX_TYPE=docker` in `server/.env`, agent sandboxes run as local Docker containers instead of MorphCloud VMs:

```bash
docker build -f docker/Dockerfile -t druids-base .    # build agent base image (required once)
```


## Workflow

### Git

Do not commit directly to `main`. Instead, create a new branch, push it, and open a PR. Use `gh pr create` for PRs. Branch naming: descriptive kebab-case (e.g. `verify-bot`, `improve-review-prompts`).

### Verification

This is non-negotiable. When opening a PR, you must demonstrate that your change actually works end-to-end. Not "it should work" — you ran it, you saw it work, and you can show your receipts.

What this means in practice:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulcrumresearch/druids](https://github.com/fulcrumresearch/druids) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
