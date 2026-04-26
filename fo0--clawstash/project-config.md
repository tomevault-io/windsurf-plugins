---
trigger: always_on
description: > **Session-Start:** Read `MEMORY.md` first to restore context from previous sessions.
---

# CLAUDE.md — Project Guide

> **Session-Start:** Read `MEMORY.md` first to restore context from previous sessions.
> After every implementation, follow the review process in `agent_docs/review_process.md`.
> Unresolved findings go to `BACKLOG.md` as defined in `agent_docs/backlog_process.md`.
> Session-spanning context and learnings go to `MEMORY.md` as defined in `agent_docs/memory_process.md`.
> **Diagram generation:** When the user requests an architecture diagram, follow `agent_docs/diagram_prompt.md`.
>   Write result to `docs/ARCHITECTURE.mmd` and generate `docs/ARCHITECTURE.svg`.
> **On "done" / "fertig":** Commit uncommitted changes (if any), and if the work relates to a GitHub
>   issue, comment on it (in English) with a summary and close it. Do NOT push unless explicitly asked.
> **On GitHub issue work:** Reference the issue number in commit messages (e.g. `fix: resolve crash #42`).

## Project Overview

**ClawStash** is an AI-optimized stash storage system, built specifically for AI agents with REST API, MCP (Model Context Protocol) support, and a web GUI.

**Core features:**
- Text and file storage with multi-file support per stash
- Name + Description: Separate title and AI-optimized description per stash
- Archive: Hide stashes from default listings without deleting (toggle in UI, API, and MCP)
- REST API for programmatic access with Bearer token auth
- MCP Server for direct AI agent integration (Streamable HTTP + stdio)
- Web dashboard with dark-theme GUI (card/list view)
- Tags (Combobox), Metadata (Key-Value Editor), Full-text search
- Access log tracking via API, MCP, and UI
- Admin login gate with session management
- Settings area with API management, token CRUD, and storage statistics
- Version history with diff comparison and restore functionality
- Mobile-optimized responsive layout with collapsible sidebar

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | TypeScript (strict mode) | 5.7 |
| Framework | Next.js (App Router) | 16 |
| Frontend | React | 19 |
| Database | SQLite (better-sqlite3) | 12 |
| MCP Server | @modelcontextprotocol/sdk | 1.27 |
| Validation | Zod | 3.24 |
| Code Editor | react-simple-code-editor, PrismJS | 0.14, 1.30 |
| Markdown Rendering | marked | 17 |
| Text Diffing | diff (jsdiff) | 8 |
| Module System | ESM (`"type": "module"`) | — |
| Containerization | Docker (multi-stage, standalone output) | — |
| CI/CD | GitHub Actions → GHCR | — |
| Linter/Formatter | — (not configured) | — |
| Test Framework | — (not configured) | — |

## Project Structure

```
clawstash/
├── package.json                # Dependencies and scripts
├── tsconfig.json               # TypeScript config (strict, ES2022, Next.js plugin, @/* path alias)
├── next.config.ts              # Next.js config (standalone output, better-sqlite3 external)
├── Dockerfile                  # Multi-stage Docker build (Node 22-slim, Next.js standalone)
├── docker-compose.yml          # Docker Compose deployment
├── .env.example                # Environment variables template
├── BACKLOG.md                  # Deferred review findings tracker
├── MEMORY.md                   # Session-spanning project knowledge
├── agent_docs/                 # Agent process documentation
│   ├── review_process.md       # Mandatory review process after every implementation
│   ├── backlog_process.md      # Backlog tracking rules and format
│   ├── memory_process.md       # Memory tracking rules and format
│   ├── refactoring_guidelines.md  # Refactoring principles and rules
│   └── diagram_prompt.md       # Architecture diagram generation instructions
├── docs/                       # User-facing documentation (split from README)
│   ├── api-reference.md        # REST API endpoints, examples, query parameters
│   ├── mcp.md                  # MCP tools, token-efficient patterns, transport options
│   ├── deployment.md           # Docker, production, CI/CD, GHCR setup
│   ├── authentication.md       # Admin login, API tokens, scopes, security
│   └── openclaw-onboarding-prompt.md  # Copy-paste onboarding prompt for OpenClaw agents
├── .github/
│   └── workflows/
│       └── docker-publish.yml  # CI: Type-check, build, push to GHCR
├── scripts/
│   └── generate-build-info.js  # Prebuild script: generates build metadata (git branch, commit, date)
├── public/                     # Next.js static assets
├── src/
│   ├── middleware.ts            # Next.js middleware (CORS, security headers, login rate limiting)
│   ├── app/                    # Next.js App Router
│   │   ├── layout.tsx          # Root layout with metadata + global CSS
│   │   ├── page.tsx            # Client component wrapper for <App />
│   │   ├── [...slug]/
│   │   │   └── page.tsx        # Catch-all route for client-side routing
│   │   ├── mcp/
│   │   │   └── route.ts        # MCP Streamable HTTP endpoint (POST/GET/DELETE)
│   │   └── api/                # API Route Handlers
│   │       ├── _helpers.ts     # Shared utilities (checkScope, checkAdmin, getBaseUrl)
│   │       ├── health/route.ts # GET health check (no auth, DB status + stats)
│   │       ├── stashes/
│   │       │   ├── route.ts            # GET (list), POST (create)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fo0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
