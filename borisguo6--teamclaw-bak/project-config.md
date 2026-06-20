---
trigger: always_on
description: A multi-agent orchestration platform with visual workflow (OASIS). Create and configure agents (OpenClaw/external API), orchestrate them into Teams, build new Teams with Team Creator, and design workflows via visual canvas. Supports Team conversations, OASIS Town with living GraphRAG memory, scheduled tasks, Telegram/QQ bots, TinyFish competitor monitoring, and Cloudflare Tunnel for remote access.
---


# TeamClaw

Use this skill to install, configure, run, operate, troubleshoot, or modify TeamClaw.

This skill is now the **entrypoint**, not the whole manual. Use **progressive disclosure**:

1. Read [docs/index.md](./docs/index.md) first.
2. Open only the doc(s) needed for the current task.
3. If you need to inspect or edit code, read [docs/repo-index.md](./docs/repo-index.md).
4. Use [README.md](./README.md) for product overview and user-facing positioning, not as the canonical operator reference.

## Task Router

Read only the relevant docs:

| Task | Read First | Then Read |
|---|---|---|
| Install / configure / start TeamClaw | This file | [docs/ports.md](./docs/ports.md) only if ports or routing matter |
| Understand what TeamClaw is | [docs/overview.md](./docs/overview.md) | [README.md](./README.md) |
| Use Team Creator or build a Team from a task description | [docs/team-creator.md](./docs/team-creator.md) | [docs/build_team.md](./docs/build_team.md), [docs/example_team.md](./docs/example_team.md) |
| Understand OASIS runtime semantics, Town Mode, GraphRAG memory, or ReportAgent | [docs/oasis-reference.md](./docs/oasis-reference.md) | [docs/runtime-reference.md](./docs/runtime-reference.md), [docs/create_workflow.md](./docs/create_workflow.md) |
| Understand runtime architecture / auth / services | [docs/runtime-reference.md](./docs/runtime-reference.md) | [docs/ports.md](./docs/ports.md), [docs/repo-index.md](./docs/repo-index.md) |
| Find CLI commands | [docs/cli.md](./docs/cli.md) | `uv run scripts/cli.py <command> --help` |
| Build or edit a Team | [docs/build_team.md](./docs/build_team.md) | [docs/example_team.md](./docs/example_team.md) |
| Create or debug a workflow YAML | [docs/create_workflow.md](./docs/create_workflow.md) | [docs/example_team.md](./docs/example_team.md) |
| Configure OpenClaw integration | [docs/openclaw-commands.md](./docs/openclaw-commands.md) | [docs/build_team.md](./docs/build_team.md) |
| Configure TinyFish competitor monitoring | [docs/tinyfish-monitor.md](./docs/tinyfish-monitor.md) | [docs/runtime-reference.md](./docs/runtime-reference.md), [docs/repo-index.md](./docs/repo-index.md) |
| Debug ports / service wiring / frontend proxy routes | [docs/ports.md](./docs/ports.md) | [docs/repo-index.md](./docs/repo-index.md) |
| Refactor / deploy / rollback | [docs/migration-playbook.md](./docs/migration-playbook.md) | [docs/backend-refactor-plan.md](./docs/backend-refactor-plan.md), [docs/repo-index.md](./docs/repo-index.md) |

## Repository Indexing

Before touching code, read [docs/repo-index.md](./docs/repo-index.md). It maps:

- runtime entrypoints (`src/mainagent.py`, `src/front.py`, `src/time.py`, `oasis/server.py`)
- install and configuration scripts (`selfskill/scripts/run.*`, `configure.py`, `scripts/setup_apikey.*`)
- frontend files (`src/static/js/main.js`, `src/static/css/style.css`, `src/templates/*.html`)
- OASIS Town / GraphRAG files (`oasis/swarm_engine.py`, `oasis/graph_memory.py`, `oasis/forum.py`)
- Team / workflow data paths under `data/user_files/...`
- OpenClaw, bots, tests, and verification commands

If the task is narrow, inspect only the indexed files for that area instead of scanning the whole repo.

## Agent Rules

### Zero-Question Install Flow

The goal is a **silent, non-interactive install**. The AI code CLI agent should NEVER repeatedly ask the user for `LLM_API_KEY`, `LLM_BASE_URL`, or `LLM_MODEL` during installation. Instead:

1. Run `setup` → `configure --init` → `start`. The `start` command:
   - Creates a default `.env` if missing
   - Starts all services even if LLM is not yet configured
   - Automatically warms an installed OpenClaw gateway and refreshes runtime `OPENCLAW_*` values
   - Does NOT silently import OpenClaw/Antigravity config — that's left to the user via the Setup Wizard
   - **Automatically starts Cloudflare Tunnel** for mobile remote access and prints the public URL
2. **Do NOT ask the user for LLM credentials before starting**. The frontend provides a first-login setup wizard where the user can configure API Key, select provider, and choose a model from a dropdown — all from the web UI.
3. After `start`, the console will print the public tunnel URL. **Always tell the user the mobile access URL**:
   - Local: `http://127.0.0.1:<PORT_FRONTEND>/mobile_group_chat`
   - Public (phone): `<PUBLIC_DOMAIN>/mobile_group_chat` — this is the URL to open on a phone browser
   - Tell the user: "在手机浏览器打开以下地址即可远程访问消息中心：`<PUBLIC_DOMAIN>/mobile_group_chat`"
4. The first-login flow handles everything:
   - Passwordless localhost login (no password needed)
   - If LLM is not configured, a setup wizard modal appears automatically
   - The wizard auto-detects local resources and shows import buttons:
     - 🦞 **从 OpenClaw 导入** — if OpenClaw is installed, one click to import API Key/Base URL/Model/Provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BorisGuo6/TeamClaw-bak](https://github.com/BorisGuo6/TeamClaw-bak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
