---
trigger: always_on
description: Self-hosted AI agent platform for Slack, powered by Claude. Agents live in Slack channels, connect to external tools and data sources, and execute tasks through conversation.
---

# TinyHands

Self-hosted AI agent platform for Slack, powered by Claude. Agents live in Slack channels, connect to external tools and data sources, and execute tasks through conversation.

## Quick Reference

```bash
npm run dev              # Start listener (development)
npm run build            # Compile TypeScript
npm test                 # Unit tests (Vitest)
npm run test:integration # Integration tests (testcontainers)
npm run lint             # ESLint
npm run typecheck        # tsc --noEmit
```

Production runs via PM2 with 6 processes: listener, 3 workers, sync, scheduler.

## Architecture

```
Slack (Socket Mode) → Listener (src/index.ts)
                        ├── Slash commands (/agents, /tools, /kb)
                        ├── Message events → relevance check → enqueue job
                        └── Express server (webhooks, internal KB API)
                                ↓
                        BullMQ + Redis (priority queue, rate limiting)
                                ↓
                        Workers (src/worker.ts, 3 instances)
                        └── Docker container per run → Claude Agent SDK
```

### Processes

| Process | Entry | Purpose |
|---------|-------|---------|
| Listener | `src/index.ts` | Slack events, commands, webhooks, Express server |
| Worker (x3) | `src/worker.ts` | Dequeue jobs, run agents in Docker containers |
| Scheduler | `src/scheduler.ts` | Evaluate cron triggers every 60s |
| Sync | `src/sync.ts` | KB source auto-sync (5 min), agent source sync (15 min), alerts (1 min), daily digest, connection health (30 min), auto-update — all cross-workspace |

## Project Structure

```
skills/                                             # Skill definitions (Markdown + YAML frontmatter)
templates/                                          # Agent template definitions (Markdown + YAML frontmatter)
src/
├── index.ts, worker.ts, scheduler.ts, sync.ts    # Process entry points
├── server.ts                                       # Express routes (webhooks, internal APIs)
├── config.ts                                       # Env var config
├── db/
│   ├── index.ts                                    # PostgreSQL pool, query helpers
│   └── migrations/                                 # SQL migrations (001-022)
├── queue/index.ts                                  # BullMQ queue, Redis, rate limiting
├── slack/
│   ├── index.ts                                    # Bolt app setup
│   ├── commands.ts                                 # Slash command handlers
│   ├── events.ts                                   # Message/mention handlers
│   ├── actions.ts                                  # Interactive action handlers
│   └── buffer.ts                                   # Real-time streaming to Slack
├── modules/
│   ├── agents/              # Agent CRUD, goal analyzer (Claude-powered config gen)
│   ├── access-control/      # Platform roles, agent roles, upgrade requests, permissions
│   ├── execution/           # Docker container lifecycle, Claude SDK, token tracking
│   ├── tools/               # Tool registry + integrations (see below)
│   ├── knowledge-base/      # KB entries, full-text search (tsvector + GIN)
│   ├── kb-sources/          # KB source connectors (GitHub, Drive, Zendesk, web) + parsers/ (docx, xlsx, pptx, pdf, rtf, html, plain)
│   ├── kb-wizard/           # Guided KB source setup flow
│   ├── reducto/             # Optional per-workspace Reducto integration for PDF / scanned-document parsing
│   ├── sources/             # Agent data sources (GitHub, Google Drive, memory)
│   ├── triggers/            # Trigger types: slack, linear, zendesk, intercom, webhook, schedule
│   ├── web-chat/            # Web Chat channels — password-protected public /chat/{token} pages that run an agent
│   ├── whatsapp/            # WhatsApp channels — agent over WhatsApp via Twilio (allowlisted phone numbers, reply threads)
│   ├── workflows/           # Multi-step stateful workflows (DAG of steps)
│   ├── teams/               # Multi-agent orchestration
│   ├── skills/              # Skill registry + builtins loader (reads /skills/*.md)
│   ├── self-evolution/      # Agent improvement proposals + approval
│   ├── self-improvement/    # Critique detection, prompt refinement
│   ├── self-authoring/      # Agent-created tools, code artifacts, MCPs
│   ├── model-selection/     # Runtime model override (/opus, /sonnet, /haiku)
│   ├── observability/       # Cost tracking, error rates, alerts, daily digest
│   ├── dashboard/           # Slack Home Tab metrics
│   ├── docs/                # Native documents (docs, sheets, files) — CRUD, versioning, search, storage
│   ├── database/            # Workspace-isolated tables (schema-per-workspace), CSV/XLSX/Google Sheet imports, read-only SQL runner, structured CRUD for the agent tool
│   ├── document-filling/    # Google Docs/Sheets template automation
│   ├── auto-update/         # Pull-based deploy from GitHub
│   ├── permissions/         # Tool access control (read-only vs read-write)
│   ├── connections/         # Encrypted credential storage, OAuth flows, connection modes, credential resolution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anantgarg/tinyhands](https://github.com/anantgarg/tinyhands) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
