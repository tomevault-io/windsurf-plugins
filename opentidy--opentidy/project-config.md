---
trigger: always_on
description: Open-source personal AI assistant. Manages administrative tasks via autonomous AI agent sessions (agent-agnostic: Claude Code, Gemini CLI, Copilot CLI), with hook-based guardrails and a web app. Distributed via Homebrew.
---

# CLAUDE.md

## Project: OpenTidy

Open-source personal AI assistant. Manages administrative tasks via autonomous AI agent sessions (agent-agnostic: Claude Code, Gemini CLI, Copilot CLI), with hook-based guardrails and a web app. Distributed via Homebrew.

**Repo:** `opentidy/opentidy` (GitHub org)
**Install:** `curl -fsSL https://opentidy.com/install.sh | bash`
**CLI:** `opentidy` (`opentidy setup`, `opentidy start`, `opentidy doctor`, etc.)
**License:** AGPL-3.0-only
**Specification:** `docs/specification.md`

## CRITICAL: Open-Source & Privacy

**This is a PUBLIC open-source repository. Every file, every commit, every diff is visible to everyone on the internet.**

### Zero personal information, no exceptions

- **NEVER** commit real names (except copyright holder in LICENSE/CLA), email addresses, phone numbers, physical addresses, account IDs, chat IDs, or any PII
- **NEVER** commit API keys, tokens, bearer tokens, passwords, or secrets of any kind
- **NEVER** commit URLs pointing to personal infrastructure (Cloudflare tunnels, self-hosted services, personal domains)
- **NEVER** reference specific user accounts, Telegram chat IDs, Gmail addresses, or real contacts in code, comments, configs, or fixtures
- **Fixtures and test data** must use only fictitious data: `example.com` emails, placeholder names ("Alice", "Bob"), fake IDs (`chat-123`), generic URLs
- **Config files** committed to the repo must contain only placeholders or empty values. Real config lives in `~/.config/opentidy/` (gitignored)

### Before every commit

Review the full diff. If **any** of the following appear, **stop and sanitize**:
- Real email addresses or phone numbers
- Telegram/WhatsApp chat IDs or bot tokens
- Bearer tokens, API keys, or OAuth credentials
- Personal domain names or tunnel URLs
- Real company names in fixtures (use generic alternatives)
- Paths containing usernames (use `~` or `$HOME` notation)

### License headers

Every source file (`.ts`, `.tsx`) must have this SPDX header as the first lines:

```typescript
// SPDX-License-Identifier: AGPL-3.0-only
// Copyright (c) 2026 Loaddr Ltd
```

## Commands

```bash
pnpm install                           # install all workspaces
pnpm build                             # build all packages
pnpm dev                               # dev mode (backend + web parallel)
pnpm test                              # vitest (backend)
pnpm test:e2e                          # playwright (web)
pnpm --filter @opentidy/backend test   # backend tests only
pnpm --filter @opentidy/web dev        # web dev only
pnpm --filter @opentidy/shared build   # shared types only
```

## Architecture

### 8 guiding principles

1. **Speed is not a criterion** (admin tasks, not real-time)
2. **AI agent CLI is the execution engine**: agent-agnostic (Claude Code, Gemini CLI, Copilot CLI), skills, MCP, browser, session resume
3. **Budget is not a constraint** (Claude Max, no token compromises)
4. **Intelligence lives in Claude, not the code**: backend does plumbing, Claude decides
5. **No interruption, isolated parallelism**: each task = its own session
6. **The assistant runs quietly in the background**: hybrid events + crons
7. **Quick/interactive actions = specialized tool**, not the core system
8. **Continuous improvement**: gaps.md = natural backlog

### Monorepo

```
opentidy/
├── pnpm-workspace.yaml
├── packages/
│   └── shared/              # TypeScript types, Zod schemas
├── apps/
│   ├── backend/             # Hono API, daemon, features (VSA)
│   └── web/                 # React SPA, Vite, features (VSA)
├── workspace/               # runtime: tasks, state.md, artifacts (gitignored)
└── docs/                    # public documentation
```

### Vertical Slice Architecture (VSA)

Backend and frontend follow VSA: code is organized by feature/domain, not by technical layer. Each feature directory is self-contained: route + handler + logic + colocated tests. An agent opens one directory and has full context.

**Backend** (`apps/backend/src/`):
```
features/
  tasks/           # CRUD, state.md parsing, title generation
  sessions/       # launch, stop, take-over, hand-back, post-session, crash recovery
  triage/         # webhook, mail/SMS readers, classify, route dispatch
  memory/         # CRUD, extraction agents, injection, lock
  suggestions/    # list, approve, dismiss, file parser
  ameliorations/  # list, resolve, ignore, gaps.md parser
  hooks/          # POST /api/hooks handler
  notifications/  # Telegram sender, notification store
  checkup/        # periodic sweep, workspace watcher
  terminal/       # ttyd bridge, port route
  system/         # health, reset, audit, processes, SSE events, test tasks
shared/           # cross-cutting: database, locks, dedup, spawn-agent, agents/, SSE, auth, config, paths
cli/              # CLI commands (setup/, doctor, status, logs, update, uninstall)
boot/             # periodic tasks setup (intervals, watchdog, crash recovery)
```

**Frontend** (`apps/web/src/`):
```
features/
  tasks/           # TaskCard, TaskDetail, Sidebar, StateRenderer
  sessions/       # SessionCard, SessionOutput

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/opentidy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
