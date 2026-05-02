---
trigger: always_on
description: AI-driven Perfetto analysis platform for Android performance data.
---

# SmartPerfetto Development Guide

AI-driven Perfetto analysis platform for Android performance data.

## Language

用中英文思考，用中文回答。Insight 内容必须使用中文。

## Compact Instructions

```
Tech: TypeScript strict, follow existing patterns
Dev:  tsx watch (backend) + build.js --watch (frontend) — auto-rebuild on save
Test: cd backend && npm run test:scene-trace-regression  ← MANDATORY after every change
Start: ./scripts/start-dev.sh (first-time) | ./scripts/restart-backend.sh (.env/npm changes only)
Build: cd backend && npm run build
```

## Post-change Dev Workflow

Both backend (`tsx watch`) and frontend (`build.js --watch`) auto-rebuild on file save. After code changes:
- **All .ts / .yaml changes**: Tell user to refresh the browser. No restart needed.
- **Only use `./scripts/restart-backend.sh`** for: `.env` changes, `npm install`, or tsx watch stuck.
- **Only use `./scripts/start-dev.sh`** for: first-time setup or both services crashed.
- **Default assumption**: User only refreshes browser after changes.

## Verification (done-conditions)

Every task must satisfy these before completion:

| Task Type | Done When |
|-----------|-----------|
| Any code change | `cd backend && npm run test:scene-trace-regression` passes (6 canonical traces) |
| Skill YAML change | `npm run validate:skills` passes + regression passes |
| Strategy/template .md change | `npm run validate:strategies` passes + regression passes |
| Build/type error | `npx tsc --noEmit` passes in backend/ |
| Pre-commit | Run `/simplify` on changed code |

## Health Stack

Tools used by `/health` for the code quality dashboard:

- typecheck: `cd backend && npx tsc --noEmit`
- test: `cd backend && npm run test:core`
- lint: _not configured_ (candidates: `biome` or `eslint`)
- deadcode: _not configured_ (candidate: `knip`)
- shell: _not configured_ (candidate: `shellcheck scripts/*.sh` — 6 shell scripts)

`/health` composites these into a 0-10 score and appends a snapshot to `~/.gstack/projects/Gracker-SmartPerfetto/health-history.jsonl` for trend tracking.

## Architecture Overview

```
Frontend (Perfetto UI @ :10000) ◄─SSE/HTTP─► Backend (Express @ :3000)
                │                                     │
                └───────── HTTP RPC (9100-9900) ──────┘
                                  │
                    trace_processor_shell (Shared)
```

**Core Concepts:**
- **Primary Runtime: agentv3** — Codex Agent SDK as orchestrator (20 MCP tools)
- **Deprecated Fallback: agentv2** — activated only by `AI_SERVICE=deepseek`
- Scene Classifier → scene-specific system prompts (12 scenes: scrolling/startup/anr/pipeline/interaction/touch-tracking/teaching/memory/game/overview/scroll-response/general)
- Analysis logic in YAML Skills (`backend/skills/`) — L1→L2→L3→L4 layered results
- SSE for real-time streaming

**Detailed rules by area:** See `.Codex/rules/` for backend, frontend, skills, prompts, git, and testing rules.

## Key Rules (NEVER / ALWAYS)

1. **NEVER hardcode prompt content in TypeScript** — use `*.strategy.md` / `*.template.md` (see `rules/prompts.md`)
2. **ALWAYS push perfetto submodule to `fork` remote**, never `origin` (see `rules/git.md`)
3. **ALWAYS run trace regression** after code changes (see `rules/testing.md`)
4. **ALWAYS check if file is auto-generated** before fixing build errors (see `rules/backend.md`)

## API Endpoints

**Agent (primary path):**
- `POST /api/agent/v1/analyze` — Start analysis
- `GET /api/agent/v1/:sessionId/stream` — SSE real-time stream
- `GET /api/agent/v1/:sessionId/status` — Poll status
- `POST /api/agent/v1/resume` — Resume analysis (multi-turn SDK context recovery)

**Multi-turn & interaction:**
- `GET /api/agent/v1/:sessionId/turns` — Get analysis turns
- `POST /api/agent/v1/:sessionId/respond` — Multi-turn response
- `POST /api/agent/v1/:sessionId/intervene` — User intervention
- `POST /api/agent/v1/:sessionId/cancel` — Cancel analysis
- `POST /api/agent/v1/:sessionId/interaction` — Handle interaction
- `GET /api/agent/v1/:sessionId/focus` — Get focus app
- `GET /api/agent/v1/:sessionId/report` — Get analysis report

**Scene reconstruction:**
- `POST /api/agent/v1/scene-reconstruct` — Start reconstruction
- `GET /api/agent/v1/scene-reconstruct/:analysisId/stream` — SSE stream
- `GET /api/agent/v1/scene-reconstruct/:analysisId/status` — Get status
- `POST /api/agent/v1/scene-reconstruct/:analysisId/deep-dive` — Deep dive
- `DELETE /api/agent/v1/scene-reconstruct/:analysisId` — Delete

**Supporting:** `/api/agent/v1/scene-detect-quick`, `/api/agent/v1/teaching/pipeline`, `/api/agent/v1/logs/*`, `/api/agent/v1/sessions`, `/api/traces/*`, `/api/skills/*`, `/api/export/*`, `/api/sessions/*`

## SSE Events (agentv3)

| Event | Description |
|-------|-------------|
| progress | Phase transitions (starting/analyzing/concluding) |
| agent_response | MCP tool results (SQL/Skill) |
| answer_token | Final text streaming |
| thought | Intermediate reasoning |
| conclusion | Near-terminal — SDK result arrives, conclusion text ready |
| analysis_completed | Terminal — HTML report generated (carries reportUrl) |
| error | Exceptions |

Note: agentv3 sends `conclusion` first (user sees result immediately), then `analysis_completed` follows after report generation.

## Analysis Mode (fast / full / auto)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gracker/SmartPerfetto](https://github.com/Gracker/SmartPerfetto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
