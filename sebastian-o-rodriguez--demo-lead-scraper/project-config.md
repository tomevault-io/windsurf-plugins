---
trigger: always_on
description: Turns unstructured search results into structured lead data with AI enrichment.
---

# AI Lead Scraper + Enricher

Turns unstructured search results into structured lead data with AI enrichment.

**Owner:** Sebastian Rodriguez
**Updated:** 2026-04-14

---

## Product

A small demo app that takes a search query, scrapes DuckDuckGo HTML results, enriches each result with AI, and displays structured lead data in a table.

This is a **sales demo** — part of a multi-demo package showing AI automation capabilities. Not a production scraper. Not a lead gen platform.

**Core loop:** Query -> Scrape DDG -> Parse titles + URLs -> AI enrich -> Render table

**Core transformation:** Unstructured search results → structured lead data

**Business value:** Skip the manual research loop. One query returns structured leads you can act on.

## Stack

| Layer | Tech |
|-------|------|
| App | Next.js (App Router) |
| UI | React + TypeScript + Tailwind |
| API | Next.js API route |
| Scraper | Python + requests + BeautifulSoup4 |
| AI | OpenRouter |

**No** microservices, separate backends, databases, auth, queues, background jobs, crawl depth, pagination, exports, retries infra, fancy design, full lead validation.

## Architecture

- One Next.js app, single page UI + one API route
- Python script called from API route for scraping
- OpenRouter for model enrichment (called from TypeScript)
- No database — everything in-memory per request
- No auth — open demo app

## Codebase

| Path | Purpose | Status |
|------|---------|--------|
| `app/page.tsx` | UI — query input, submit, loading/error, results table | Complete |
| `app/api/leads/route.ts` | API route — calls scraper, enriches, returns leads | Complete |
| `lib/types.ts` | RawLead, Lead, request/response types | Complete |
| `lib/enrich.ts` | OpenRouter enrichment with retry (parallel, paid model) | Complete |
| `lib/mock-leads.ts` | Fallback mock data when scraper fails | Complete |
| `scripts/scrape.py` | Fetch DDG HTML, parse titles + URLs, print JSON | Complete |

## Workspace Layout

```
ai-lead-enricher/              <- YOU ARE HERE (git root)
├── CLAUDE.md                  <- Start here, always
├── Justfile                   <- CLI entry point: just <target>
├── .claude/                   <- Agent definitions + hooks
│   ├── settings.json          <- Hooks (format-on-save, block destructive git)
│   └── agents/                <- robo, architect, backend, frontend, qa
├── .gorp/                     <- Plans, process docs, prompts, journal
│   ├── plans/                 <- current-sprint.md, roadmap.md
│   ├── process/               <- conventions, agent-protocol, approval-matrix
│   ├── prompts/               <- dispatch.md.tmpl
│   └── journal/               <- Per-agent daily logs
├── app/
│   ├── page.tsx               <- Single page UI
│   └── api/leads/route.ts     <- Lead scrape + enrich endpoint
├── lib/
│   ├── types.ts               <- Type definitions
│   ├── enrich.ts              <- OpenRouter enrichment
│   └── mock-leads.ts          <- Fallback mock data
├── scripts/
│   ├── scrape.py              <- DDG scraper
│   ├── dispatch.sh            <- Agent dispatch
│   └── quality-gate.sh        <- Quality checks
├── public/                    <- Static assets
├── .env.example               <- Environment template
├── requirements.txt           <- Python deps
└── README.md                  <- Project overview
```

## Data Model

No database. All data is ephemeral per request.

```
RawLead  { name: string, url: string }
Lead     { name: string, url: string, summary: string, category: string }
```

**RawLead** — output from Python scraper (title + URL from DDG HTML)
**Lead** — enriched by AI model with summary + category

## Views (v1)

1. **Search Page** — Single page with query input, submit button, loading spinner, error display, and results table

That's it. One page.

## UX Rules

- Single page, no navigation
- Query input + submit button at top
- Loading state while scraping + enriching
- Error state if anything fails
- Results table with 4 columns: name, url, summary, category
- 5-10 rows max
- Desktop-first, clean and readable

## Non-Goals

No: auth, database, exports, pagination, background jobs, retries, crawl depth, fancy design, full lead validation, multi-page UI, caching, rate limiting, user accounts

---

## Key Documents

| Doc | Path | Purpose |
|-----|------|---------|
| Launch roadmap | `.gorp/plans/roadmap.md` | Canonical planning reference |
| Conventions | `.gorp/process/conventions.md` | Git, commits, sprint format |
| Agent protocol | `.gorp/process/agent-protocol.md` | Dispatch/report format for agents |
| Approval matrix | `.gorp/process/approval-matrix.md` | What agents can auto-approve vs owner-required |
| Current sprint | `.gorp/plans/current-sprint.md` | Active tasks |
| Gotchas | `.gorp/process/gotchas.md` | Situational issues |

---

## Agent System

This repo uses Claude Code multiagent orchestration. Agents live in `.claude/agents/`.

| Agent | Model | Role | Scope |
|-------|-------|------|-------|
| **robo** | opus | Sprint orchestrator, dispatches others | Plans, coordination |
| **architect** | sonnet | API contracts, type design | Types, contracts |
| **backend** | sonnet | API route, enrichment, scraper integration | `app/api/`, `lib/`, `scripts/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sebastian-O-Rodriguez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
