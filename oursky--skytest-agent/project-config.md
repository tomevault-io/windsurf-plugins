---
trigger: always_on
description: ├── lib/                           # Backend domain modules + singletons
---

# AGENTS.md — Guidelines for Codex

## Project Map
```
apps/web/src/
├── lib/                           # Backend domain modules + singletons
│   ├── runtime/                   # Run lifecycle and execution
│   │   ├── test-runner.ts         # Shared execution engine
│   │   ├── local-browser-runner.ts# Browser runtime dispatch target
│   │   └── usage.ts               # API usage tracking
│   ├── runners/                   # Runner orchestration + queueing services
│   ├── android/                   # Android devices/emulators runtime
│   ├── core/                      # Shared core modules (prisma/logger/errors)
│   ├── security/                  # Authentication + security helpers
│   ├── storage/                   # Object storage adapters + helpers
│   ├── test-cases/                # Test case domain logic
│   ├── test-config/               # Test config parsing/validation/sorting
│   └── mcp/                       # MCP server/tooling
│
├── app/                           # Next.js App Router
│   ├── api/                       # REST API endpoints
│   │   ├── projects/              # Project CRUD + project configs
│   │   ├── teams/                 # Team settings, members, runners, usage
│   │   ├── runners/v1/            # Runner protocol endpoints
│   │   ├── test-cases/            # Test case CRUD + files + history + export
│   │   ├── test-runs/             # Run status, cancel, SSE events, dispatch
│   │   ├── user/                  # User settings & API keys
│   │   ├── mcp/                   # MCP transport endpoint
│   │   └── health/                # Live/ready/dependencies probes
│   ├── projects/                  # Project list & detail pages
│   ├── teams/                     # Team settings pages
│   ├── test-cases/[id]/           # Test case history views
│   └── run/                       # Main test runner page
│
├── components/                    # React components (feature-first)
│   ├── features/
│   │   ├── test-builder/          # Test builder + step editing
│   │   ├── test-configurations/   # Test-level config composition
│   │   ├── project-configurations/# Project-level config management
│   │   ├── test-files/            # File upload/list widgets
│   │   ├── run-results/           # Run timeline + status + artifacts
│   │   ├── test-cases/            # Test case list/detail UI
│   │   ├── projects/              # Project list/detail UI
│   │   ├── team-runners/          # Runner inventory + troubleshooting UI
│   │   ├── team-members/          # Team membership UI
│   │   ├── team-usage/            # Team usage UI
│   │   └── team-ai/               # Team AI key/settings UI
│   ├── shared/                    # Cross-feature reusable UI
│   └── layout/                    # Page-level layout primitives
│
├── workers/                       # Long-running maintenance/dispatch loops
│   ├── runner-maintenance.ts
│   └── browser-runner.ts
│
├── types/                         # TypeScript interfaces
│   └── index.ts                   # All type exports
│
├── config/app.ts                  # App configuration
└── i18n/messages.ts               # i18n keys (en/zh-Hant/zh-Hans)
```

## Task Routing

| Task | Start Here | Related Files |
|------|------------|---------------|
| Fix test execution | `apps/web/src/lib/runtime/test-runner.ts` | `apps/web/src/lib/runtime/local-browser-runner.ts`, `apps/macos-runner/runner/index.ts` |
| Fix browser run dispatch | `apps/web/src/lib/runtime/browser-run-dispatcher.ts` | `apps/web/src/workers/browser-runner.ts`, `apps/web/src/app/api/test-runs/dispatch/route.ts` |
| Fix run scheduling/claiming | `apps/web/src/lib/runners/claim-service.ts` | `apps/web/src/app/api/runners/v1/jobs/claim/route.ts` |
| Fix runner event ingestion | `apps/web/src/lib/runners/event-service.ts` | `apps/web/src/app/api/runners/v1/jobs/[id]/events/route.ts` |
| Fix SSE/real-time updates | `apps/web/src/app/api/test-runs/[id]/events/route.ts` | `apps/web/src/components/features/run-results/ui/ResultViewer.tsx` |
| Fix test case CRUD | `apps/web/src/app/api/test-cases/` | `apps/web/src/types/test.ts`, `apps/web/src/lib/test-cases/` |
| Fix project CRUD/configs | `apps/web/src/app/api/projects/` | `apps/web/src/lib/core/prisma.ts` |
| Fix team runners/members/usage | `apps/web/src/app/api/teams/` | `apps/web/src/components/features/team-runners/`, `apps/web/src/components/features/team-members/`, `apps/web/src/components/features/team-usage/` |
| Fix authentication | `apps/web/src/lib/security/auth.ts` | `apps/web/src/app/api/`, `apps/web/src/lib/runners/auth.ts` |
| Fix MCP tooling | `apps/web/src/lib/mcp/` | `apps/web/src/app/api/mcp/route.ts` |
| Change DB schema | `apps/web/prisma/schema.prisma` | `apps/web/src/types/`, `apps/web/src/lib/core/prisma.ts` |

## Tech Stack
- Next.js 16 (App Router), React 19, TailwindCSS 4
- Prisma + PostgreSQL, Server-Sent Events
- Playwright 1.57, Midscene.js

## Docs To Read First
- `CLAUDE.md` - Complementary AI coding guidelines and expanded project map
- `docs/README.md` - Documentation index and audience split
- `infra/README.md` - Local infra topology and shared deployment dependencies
- `docs/maintainers/coding-agent-maintenance-guide.md` - Runtime invariants and common footguns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oursky/skytest-agent](https://github.com/oursky/skytest-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
