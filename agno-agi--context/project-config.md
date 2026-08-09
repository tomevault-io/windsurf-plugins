---
trigger: always_on
description: This file is the source of truth for any agent (Claude Code, Codex, others) working in this repo. `CLAUDE.md` is a symlink to this file — edit one, both update.
---

# @context

This file is the source of truth for any agent (Claude Code, Codex, others) working in this repo. `CLAUDE.md` is a symlink to this file — edit one, both update.

## What this is

`@context` is a self-hosted **context agent** — a professional alter-ego you own. One [Agno](https://docs.agno.com) agent that **captures, files, and retrieves** your working context across many sources, and that other people (and their agents) can leave updates with. **Anyone can write to your context. Only you can read it** — or act through it.

Two ideas define it:

- **The tool surface is the job description, not the union of every API.** Each source is a [ContextProvider](https://ashpreetbedi.com/context-providers): a sub-agent behind at most two tools — `query_<id>` (read) and `update_<id>` (write). The main agent sees `2N` tools for `N` sources, and each source's quirks stay inside its own sub-agent's scope.
- **The asymmetry is the security model.** The toolset is chosen *in code, from a verified identity, before the model runs*: the **owner** gets the full surface; **everyone else** gets the capture surface — `submit_update` (append to the owner's queue, no readback), `my_updates` (read back *only their own* submissions — the filter is the verified caller identity, closed over in code), and, when the calendar is connected, `owner_availability` (open windows from the free/busy API — intervals only, never event contents). A guest never holds a read tool into the owner's data, so "don't leak the owner's data" is structural, not a prompt rule. (One deliberate exception rides outside the toolset: per-user learning — @context remembers details about *whoever* talks to it (memory + profile), scoped to that caller's own identity, never the owner's data. See [`docs/SECURITY.md`](docs/SECURITY.md).)

## Architecture

```
Context  (agents/context.py — one Agno agent, gpt-5.6-sol)
│
├── ContextProviders (agents/sources.py)        each source = query_<id> / update_<id>
│   ├── crm        DatabaseContextProvider        structured store (crm schema)  R/W  always on
│   ├── knowledge  WikiContextProvider            knowledge base — specs (FS → Git)  R/W  always on
│   ├── workspace  WorkspaceContextProvider       this repo's files                  R    always on
│   ├── web        WebContextProvider             Parallel (SDK or keyless MCP)      R    always on
│   ├── slack      SlackContextProvider           channel / DM history; send = update_slack (ungated)  R/W  SLACK_BOT_TOKEN set
│   ├── gmail      GmailContextProvider           inbox; update_gmail drafts only     R/W  GOOGLE_* set
│   └── calendar   GoogleCalendarContextProvider  events; write = act tool (approval) R/W* GOOGLE_* set
│        (*act tool — update_calendar — pauses for per-call owner approval. update_gmail only drafts (never sends), update_slack is ordinary messaging — both ungated.)
│
├── Inbound queue (agents/inbox.py)             submit_update + my_updates (everyone) / rundown + acknowledge (owner; acks DM the submitter a receipt)
│
├── Guest scheduling (agents/scheduling.py)     owner_availability — free/busy windows only (guest-held, iff GOOGLE_* set)
│
├── Workflows (workflows/ → WORKFLOWS)          runnable Agno Workflow objects (registered with AgentOS), owner-only
│   ├── reminders (workflows/reminders.py)      hourly sweep: queue_reminders → inbound queue  (+ the queue_reminders owner tool)
│   ├── digest    (workflows/digest.py)         daily rundown / weekly week-plan → owner Slack DM (auto-armed when SLACK_BOT_TOKEN set)
│   └── notify    (workflows/notify.py)         dm_owner() / dm_user() — proactive DMs: sweep + digests to the owner, ack receipts to submitters
│
├── Schedules (app/schedules.py)                register_schedules() — cron that fires the workflows (hourly sweep; Slack-gated digests)
│
├── Skills (skills/ + agents/policy.py)         owner-only playbooks  week-plan / daily-rundown / prep-for / process-today / research / knowledge-review
│
├── MCP server (app/mcp.py)                     owner-only `use_context` at /mcp (on by default) — read/act/file via Claude/ChatGPT desktop + CLI
│
└── Owner policy (agents/policy.py + app/identity.py)
    the identity-conditioned surface (instructions + toolset) + pre-hook + tool-hook — all from a verified id
```

Shared:
- PostgreSQL + pgvector for sessions, memory, knowledge, and the `crm` schema (the structured store).
- `app.settings.default_model()` returns `OpenAIResponses(id="gpt-5.6-sol")` — bump the model in one place.
- Scheduler enabled by default (`scheduler=True`). Scheduled runs arrive with the verified identity `__scheduler__`, which `is_owner` treats as the owner (the scheduler is the owner's automation — see `docs/SECURITY.md`).
- Slack interface is added automatically when both `SLACK_BOT_TOKEN` and `SLACK_SIGNING_SECRET` are set, routed to `context` ([`docs/SLACK.md`](docs/SLACK.md)).
- JWT auth on whenever `RUNTIME_ENV == "prd"`, with `user_isolation=True` (so production deploys are gated by default).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agno-agi/context](https://github.com/agno-agi/context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
