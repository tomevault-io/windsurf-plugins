---
trigger: always_on
description: > **Fill this section in for your project.** Everything below (trigger table + always-on rules + directory structure) is project-agnostic.
---

# Project Placeholder — _<Your Project>_

> **Fill this section in for your project.** Everything below (trigger table + always-on rules + directory structure) is project-agnostic.

_One-paragraph product description goes here. What you're building, who for, what competitors you'll displace._

## Architecture (fill in)
- **Monorepo / single repo:** _(turborepo+pnpm / single-package / other)_
- **Backend:** _(NestJS / FastAPI / Rails / …)_
- **Frontend:** _(Next.js / SvelteKit / …)_
- **Shared contracts:** _(Zod in @your-project/shared / OpenAPI / …)_

## Quick Start (fill in)
```bash
# cp .env.example .env
# <deps install>
# <db setup>
# <run dev>
```

## Commands (fill in)
| Command | Description |
|---------|-------------|
| _(project build/lint/test/typecheck commands)_ | _(descriptions)_ |

## Task Management (TaskMaster)

Canonical task source for all features, bugs, and backlog items. Run `npx task-master --help` for the full command list; `next` / `list` / `show <id>` / `set-status --id=<id> --status=<status>` / `add-task --prompt="<desc>"` / `expand --id=<id>` are the common ones.

## Test Users (fill in)
_(local dev + prod canonical test accounts — never commit passwords)_

---

# ⚡ Trigger Table — READ THESE FILES WHEN:

**This is the most important section of this file. Each row is a conditional instruction: when the trigger fires, you MUST read the linked file BEFORE acting.**

| Trigger | READ BEFORE acting |
|---|---|
| **Starting a NEW project** (no prior waves; seeding from founder docs) | `command-center/rules/onboarding/onboarding-loop.md` (13-stage pre-launch sequence v0→v11; hands off to wave-loop Stage 0 at the end) |
| Starting a new wave | `command-center/rules/build-iterations/wave-loop.md` (then read each stage file before entering that stage) |
| Picking next task / checking backlog | `npx task-master next` or `npx task-master list` — TaskMaster is the canonical task source |
| Spawning ANY sub-agent | `command-center/rules/sub-agent-workflow.md` + `command-center/Sub-agent Instructions/<agent-name>-instructions.md` |
| Any test work (Playwright, Vitest, UI verification, prod audit) | `command-center/rules/testing-principles.md` + `command-center/test-writing-principles.md` §15-16 + `command-center/artifacts/user-journey-map.md` |
| Making a product/UX decision | `command-center/management/semi-assisted-mode.md` (3-tier autonomy) + `command-center/management/full-autonomy-mode.md` (BOARD routing under full-autonomy) |
| Authoring / editing a milestone, changing `roadmapMilestone` metadata on a task, walking the unassigned queue | `command-center/rules/roadmap-lifecycle.md` (schema, states, edit permissions, reference format) |
| Founder says "refresh the roadmap" / "re-plan" / "strategic review" — OR triggered by Stage 11 when `planned` milestones drop below 3 — OR triggered by Stage 0b when unassigned-queue count > 30 | `command-center/rules/roadmap-refresh-ritual.md` (heavyweight milestone-level refresh; propose, do not auto-fire) |
| Founder says "daily checkpoint" / "checkpoint" / "what's pending?" — OR triggered by Stage 11 when `task-master next` returns nothing actionable AND any checkpoint bucket is non-empty | `command-center/rules/daily-checkpoint.md` (3-bucket batch: Tier 3 / assigned this cycle / stayed unassigned) |
| Wave touches auth / payments / user creation / cookies / CSRF / rate limits / sessions | `command-center/rules/security-waves.md` |
| Creating a `MONITOR:` task for any external wait (deploy, CI, DNS, tier activation, third-party provisioning) | `command-center/rules/monitors/monitor-principles.md` + platform template in `command-center/rules/monitors/` (`railway-deploy.md` / `gh-actions.md` / `netlify-deploy.md` / etc.). **Every monitor MUST declare all three of `success_condition`, `failure_condition`, `timeout_budget` — a monitor with only a success check will sit forever on a failed deploy.** |
| Task touches any external SDK or third-party tool | `command-center/rules/external-sdks.md` (pre-build checklist + SDK registry) |
| Stage 3b — design-gap resolution (UI/icon/page/flow not in `design/`) | `command-center/rules/build-iterations/stages/stage-3b-design-gap.md` (formal Dx gate between 3 and 4; skip for non-UI waves; non-blocking bugs routed to `bug-design` TaskMaster tag) |
| User says "run overnight" / "autonomously" / "I'm going to sleep" (or reverse: "I'm back" / "pause") | `command-center/management/mode-switching.md` (flag spec + transitions) → sets `mode: semi-assisted` by default. See `command-center/management/semi-assisted-mode.md` for behavior. |
| User says "full autonomy" / "go completely autonomous" / "board mode" / "unconditional loop" | `command-center/management/mode-switching.md` (sets `mode: full-autonomy`) + `command-center/management/full-autonomy-mode.md` (BOARD routing + `/loop` bootstrap + STATUS file tick behavior) + `command-center/management/board.md` + `command-center/management/board-members.md` + `command-center/management/conflict-resolution.md`. **Agent bootstraps `/loop` skill on mode entry; routes each tick via `command-center/management/STATUS`. Founder does not invoke `/loop` manually.** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [claudomat-dev/claudomat-mini](https://github.com/claudomat-dev/claudomat-mini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
