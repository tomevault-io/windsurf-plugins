---
trigger: always_on
description: **Generated:** 2026-04-09
---

# Colink — Project Knowledge Base

**Generated:** 2026-04-09  
**Commit:** 72baf2f  
**Branch:** master

## Overview

Multi-agent software development platform (formerly ISDP). Go backend (Gin + raw SQL) orchestrates Claude CLI, OpenCode CLI, and ACP protocol adapters to run AI agents in workflows. React frontend (Ant Design + Zustand) provides the workbench UI. Electron installer packages Setup and Launcher as dual Windows apps. Feishu (Lark) IM integration enables users to interact with agents via chat.

## Structure

```
isdp/                              # Repository root (Go module root — go.mod here)
├── cmd/server/                    # Main entry point (main.go, ~970 lines)
├── cmd/migrate_mysql/             # DB migration utility
├── internal/
│   ├── api/                       # 22 Gin HTTP handlers
│   ├── config/                    # Logging setup (Zap + Lumberjack)
│   ├── middleware/                 # Invite code auth
│   ├── model/                     # 19 domain entity files
│   ├── parser/                    # Mention parser
│   ├── repo/                      # 33 repository files (raw SQL)
│   ├── service/                   # 20 service packages (business logic)
│   │   ├── agent/                 # Orchestrator + adapters (Claude/OpenCode/ACP) + ChunkListener
│   │   ├── a2a/                   # Agent-to-Agent routing + queue
│   │   ├── im/                    # Feishu (Lark) IM bridge service
│   │   ├── sandbox/               # Docker/local process execution
│   │   ├── workflow/              # Workflow template CRUD
│   │   ├── configgen/             # Per-agent config directory generation
│   │   ├── teampackage/           # Team package import/export
│   │   ├── assetpackage/          # Asset package import/export
│   │   ├── skill/                 # Skill CRUD + federated registry
│   │   └── knowledge/             # Knowledge base CRUD
│   └── ws/                        # WebSocket hub
├── pkg/config/                    # YAML config loader (Viper)
├── web/                           # React frontend (see web/AGENTS.md)
├── sql-change/                    # DB migrations (see sql-change/AGENTS.md)
├── configs/                       # Config template + local config
├── .devcontainer/                 # DevContainer (Go 1.25 + Node 20 + MySQL + Redis)
├── installer/                     # Electron installer (see installer/AGENTS.md)
├── docs/                          # Plans, specs, changelog
│   ├── superpowers/plans/         # Implementation plan docs
│   ├── superpowers/specs/         # Design spec docs
│   └── CHANGELOG.md               # Dev history
├── auto-test/                     # Unified test directory (340 cases)
│   ├── e2e/                       # Playwright E2E tests
│   ├── internal/                  # Go unit tests (imports internal/)
│   ├── vitest/                    # Vitest component tests
│   └── feature-map.yaml           # Feature ID → test mapping
├── scripts/                       # Test runners, utilities
├── VERSION                        # Base version: 1.0.0
├── Makefile                       # build, run, test, clean
└── CLAUDE.md                      # AI guidance (naming, config, DB rules)
```

## Where to Look

| Task | Location | Notes |
|------|----------|-------|
| Add API endpoint | `internal/api/` | One handler file per resource; register routes in `cmd/server/main.go` |
| Add service logic | `internal/service/<domain>/` | New package per domain; inject via `main.go` |
| Add DB table/column | `sql-change/migrations/` | Create migration file, update model + repo |
| Add frontend page | `web/src/pages/` | Add route in `App.tsx`, API method in `api/client.ts` |
| Add frontend component | `web/src/components/` | Ant Design components; use Zustand for state |
| Change agent execution | `internal/service/agent/` | Adapter interface in `adapter.go` |
| Add ACP-based adapter | `internal/service/agent/acp_adapter.go` + `adapter.go` | Extend `BaseACPAdapter`, register in `NewAdapter()` |
| Change A2A routing | `internal/service/a2a/` | `EnqueueA2ATargets()` in `a2a_trigger.go` |
| Change Feishu IM | `internal/service/im/` | `FeishuBridgeService` → `LarkCLIClient` → lark-cli |
| Add IM platform | `internal/service/im/` + `model/im_session.go` + `api/feishu_webhook_handler.go` | Follow Feishu pattern: handler → bridge service → chunk forward |
| Modify installer | `installer/src/main/` | `index.ts` = Setup, `launcher-entry.ts` = Launcher |
| Update config schema | `pkg/config/config.go` + `configs/config.yaml.example` | Both files must sync |
| Run backend | `.` | `make run` or `go run ./cmd/server` |
| Run frontend dev | `web/` | `npm run dev` (port 3000, proxies to 8080) |
| Run tests | `.` | `make test` (Go); `cd web && npm run test:e2e` (Playwright) |
| Run auto-tests | `auto-test/` | `make test-all`; `python scripts/run-feature-tests.py --feature F001` |
| Build release | `installer/` | `./build.sh` (Unix) or `.\build.ps1` (Windows) |
| DevContainer | `.devcontainer/` | MySQL 8 + Redis 7 + Go 1.25 + Node 20 + Playwright |

## Code Map

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `Orchestrator` | struct | `service/agent/orchestrator.go` | Central agent dispatch; spawns agents, coordinates A2A |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lv-stupidboy/Colink](https://github.com/lv-stupidboy/Colink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
