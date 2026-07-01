---
trigger: always_on
description: Personal AI agent platform: React web UI + FastAPI backend + async worker, deployed as
---

# y-agent

Personal AI agent platform: React web UI + FastAPI backend + async worker, deployed as
AWS Lambda (SAM). Runs Claude Code / Codex subprocesses remotely on EC2 over SSH, with
a Telegram bot surface and cross-skill orchestration via trace context.

## Architecture

```
Web (React)  ─┐                                ┌─→ Claude Code / Codex subprocess (EC2, SSH)
Telegram Bot ─┼─→ API (FastAPI/Lambda) → SQS → Worker (Lambda) ─┤
CLI (y)      ─┘                                                 └─→ Post-hooks (trace, telegram, todo)

Storage (SQLAlchemy / PostgreSQL) is shared by API / Worker / CLI / admin.
```

## Packages

UV workspace with Python members + one React frontend:

| Package | Purpose | Entry |
|---------|---------|-------|
| **storage** | ORM models, repos, services, DTOs, celery config, global config loader | `src/storage/` |
| **agent** | Claude Code / Codex runners, SSH/EC2 pool, tool shims, skills discovery | `src/agent/claude_code.py`, `src/agent/codex.py` |
| **api** | FastAPI REST + SSE, JWT auth, controllers for each feature | `src/api/app.py` (port 8001) |
| **worker** | Celery/SQS task consumer, runs agent subprocesses, post-hooks, RSS pipeline | `src/worker/runner.py` |
| **cli** | Click CLI (`y` command), all feature subcommands | `src/yagent/command_option.py` |
| **admin** | Lambda handler for DB init + scheduled jobs (reminders, RSS) | `handler.py` |
| **web** | React 19 + Vite + TailwindCSS SPA | `src/main.tsx` (port 5174) |

Other top-level dirs: `scripts/` (deploy, DNS, IAM), `template.yaml` / `samconfig.toml`
(SAM), `worktree/post-create.sh` (symlink shared files into new dev worktrees).

## Tech Stack

- **Python 3.11+**, UV workspace, Hatchling build
- **FastAPI** + Uvicorn + SSE (sse-starlette) + Lambda Web Adapter (response streaming)
- **SQLAlchemy 2.0** + PostgreSQL (psycopg v3), DynamoDB (per-process lease cache)
- **Celery 5.3** (filesystem broker local, SQS in prod)
- **React 19** + React Router 7 + Vite 7 + TailwindCSS 4 + SWR
- **AWS SAM**: Lambda (API / Worker / Admin), SQS, S3 + CloudFront (web + link/RSS content),
  EventBridge schedules (reminders, RSS), DynamoDB
- **Integrations**: Telegram Bot API, Google OAuth, SSH/Paramiko, EC2 lifecycle (boto3),
  oxylabs (WeChat / generic pages), yt-dlp (YouTube), Jina AI reader (Twitter/X)

## Notable Subsystems

These are the cross-cutting features to be aware of before touching the code. Each has
entity + controller + service + CLI slices, and most have a web panel.

- **Trace** — every notify / chat / worker step carries a `trace_id` and optional
  `from_chat` / `from_topic`. Participants are registered in `run_chat`; TraceView renders
  the waterfall. `trace_share` makes a trace publicly viewable (optionally with a password).
- **Notify (cross-skill)** — `/api/chat/notify` and `y chat -m "..."` (fire-and-forget,
  default top-level mode) dispatch a message to a topic (skill). Default target is
  the DM (manager). Trace/from meta is attached on send; short-circuited callbacks
  back to root topics never invoke the LLM.
- **Topic** — every chat has an optional `topic` (named persistent address). The
  conventional root topic is `manager`; the API rejects notify callbacks aimed at
  root topics (they are conversations, not function calls).
- **Note** — `note` + `note_todo_relation` tables. A note has a `content_key` file
  pointer (relative to Y_AGENT_HOME) plus JSON `front_matter`; used for plan /
  requirement / decision / journal context tied to todos.
- **Entity (knowledge graph)** — `entity` + `entity_note_relation` + `entity_rss_relation`.
  Web sidebar exposes entities as a first-class panel.
- **RSS** — two-stage pipeline: admin schedules feed jobs → worker scrapes feed XML →
  downloader fetches each item's content → storage on S3 (per-activity key). `y rss` CLI
  for feeds + items.
- **Link archive** — EC2 is the single source of truth: `~/luohy15/links/<link_id>/{content,summary}.md` is canonical, `content_key`/`summary_content_key` are paths relative to `~/luohy15/` on EC2, API reads via SSH-cat, and S3 is not used for links.
- **Browser cookies** — `y cookies sync` stores local browser cookies in the API/DB so remote `y link fetch` can pass them to `yt-dlp`.
- **Reminder** — `reminder` table, `/api/reminder`, `y reminder` CLI. Admin Lambda runs
  `check_reminders` on a schedule and pushes matches to Telegram.
- **Telegram** — forum topic binding (`tg_topic`), webhook secret verification,
  markdown → HTML conversion, per-topic routing, root-topic callbacks short-circuited
  at the API layer. Web-only artifact fences are stripped to `[chart]` / `[diagram]`
  / `[svg]` placeholders before Telegram delivery.
- **Artifacts** — assistant markdown fences tagged `mermaid`, `vega-lite`, or
  `artifact-svg` render inline in `MessageBubble` via lazy Mermaid / Vega-Lite / sanitized
  SVG rendering. Plain `svg` fences remain code blocks.
- **Image transport** — API image ingestion stores bytes only under
  `/Users/roy/luohy15/assets/images/`: local writes when available, otherwise SSH-push
  to EC2. Workers SSH-fetch local EC2 paths before Telegram delivery. `Message.images`
  may contain EC2 asset paths or deliberate `http(s)://` pass-through URLs, never new

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luohy15/y-agent](https://github.com/luohy15/y-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
