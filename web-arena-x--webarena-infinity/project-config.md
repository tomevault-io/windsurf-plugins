---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

WebArena-Infinity is a scalable pipeline for auto-generating web-app testing environments and evaluating AI browser agents (Gemini, GPT, Claude) against them. The pipeline: generates HTML/CSS/JS apps from documentation using Claude Code → runs browser agents on task suites → audits results and iterates.

## Commands

### Local Evaluation

```bash
# Single task (starts server automatically, results go to apps/<app>/results/)
python evaluation/run_eval_parallel.py --model gemini-pro --task-id task_e1 --workers 1 --web-app apps/linear-account-settings

# Filter by difficulty
python evaluation/run_eval_parallel.py --model gpt --difficulty easy --workers 4 --web-app apps/linear-account-settings

# Parallel evaluation against remote pre-running servers
python evaluation/run_eval_parallel.py --model gemini-pro --workers 8 \
    --env-host ec2-host --base-port 8001 --web-app apps/linear-account-settings
```

### Running an App Server Directly

Servers use `SimpleHTTPRequestHandler` and serve files relative to CWD, so you must `cd` into the app directory first:

```bash
cd apps/gmail && python3 server.py --port 8000
```

### AWS Pipeline

Each environment runs as a self-contained pipeline on one EC2 instance (no SQS, no cross-machine coordination).

```bash
# Launch one EC2 instance per environment in manifest
bash infra/setup/launch.sh --manifest infra/env_manifest.jsonl --model gemini-pro

# SSH into each instance, then:
#   claude login
#   claude plugins install frontend-design
#   nohup python infra/pipeline.py --app-name <env_id> --docs-path <docs> \
#     --model gemini-pro --workers 8 > /tmp/mirror-mirror-logs/pipeline.log 2>&1 &

# Monitor progress
bash infra/setup/monitor.sh

# Collect results from all branches
python infra/collect_results.py

# Tear down
bash infra/setup/teardown.sh --release-eips
```

### Package Management

Uses `uv` (not pip). Python >=3.12 required. The single dependency is `browser-use>=0.11.9`. Shared venv lives at `~/mirror-mirror/.venv`. Run `bash setup.sh` from the repo root to install everything (Python deps, Playwright Chromium, and OS-level browser dependencies).

## Architecture

### Pipeline Data Flow

Each environment runs independently on one EC2 instance via `infra/pipeline.py`:

```
pipeline.py (one instance per environment)
├─ Phase 1: Generate App (Claude CLI)
│   └─ Writes app code + APP_DESCRIPTION.md
├─ Phase 2: Function Tasks
│   ├─ 2a: Generate function tasks (Claude CLI, once)
│   │   └─ Sanity check (fix if needed) → commit
│   └─ 2b: Eval-Audit loop (up to max_iterations)
│       └─ eval → audit failures (Claude) → sanity check → commit
├─ Phase 3: Real Tasks
│   ├─ 3a: Generate real tasks (Claude CLI, once)
│   │   └─ Sanity check (fix if needed) → commit
│   └─ 3b: Eval-Audit loop (up to max_iterations)
│       └─ eval → audit failures (Claude) → sanity check → commit
├─ Phase 4: Task Hardening (N rounds, --hardening-rounds)
│   └─ Per round:
│       ├─ 4a: Analyze agent behavior + generate harder tasks (Claude)
│       │   └─ Reads history.json from results/, appends to real-tasks.json
│       │   └─ Sanity check (fix if needed, revert if irrecoverable) → commit
│       └─ 4b: Eval-Audit loop (new tasks only, via --task-id filter)
│           └─ eval → audit failures (Claude) → sanity check → commit
└─ Phase 5: Final Regression Eval
    └─ Full-suite eval on function tasks + real tasks (no audit)
```

### Environment Protocol (every app must follow)

Each app exposes a standard HTTP API consumed by the evaluation harness:

- `GET /api/state` — returns current app state JSON (read by verifiers)
- `PUT /api/state` — browser pushes state on every mutation
- `POST /api/reset` — restores seed state, sends SSE reset event
- `GET /api/events` — SSE stream for reset notifications
- `GET /*` — static file serving

**State sync contract:** Browser PUTs full state on first load → server captures as immutable `_seed_state` → browser PUTs on every mutation → verifiers read via GET → reset restores seed state and sends SSE event to browser.

### App Structure (each app under `apps/`)

```
{app-name}/
├── server.py          # HTTP server implementing the protocol above
├── index.html         # Entry point
├── js/                # app.js, state.js, views.js, components.js, data.js
├── css/styles.css
├── real-tasks.json    # 24 tasks: 8 easy, 8 medium, 8 hard
├── real-tasks/        # Verifier scripts (task_e1.py .. task_h8.py)
├── sanity_check.py    # Automated verifier validation
├── Dockerfile
└── results/           # Evaluation output per model run
```

### Verifier Pattern

Each `real-tasks/task_*.py` exports `verify(server_url: str) -> tuple[bool, str]`. Verifiers read `/api/state` and check conditions — they never interact with the UI.

### Key Design Constraints for Apps

- No native OS UI elements (`<select>`, `alert()`, `confirm()`, file pickers) — use custom JS-rendered equivalents
- Rich realistic seed data (10+ items per dropdown, varied formats)
- Form validation with required fields and conditional requirements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [web-arena-x/webarena-infinity](https://github.com/web-arena-x/webarena-infinity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
