---
trigger: always_on
description: This file is the operational source-of-truth for AI coding assistants working on ClawTrol.
---

# AGENTS.md - ClawTrol Operational Guide

This file is the operational source-of-truth for AI coding assistants working on ClawTrol.

## 1) Operational Context (Critical)

Use this map first. Most recent confusion came from editing the wrong folder.

### Canonical vs mirror folders

**UPDATED 2026-04-19** — Workflow shifted from SSH-edit-on-VM to local-clone + deploy-to-VM. The Windows local dir now holds a FULL git clone, not a docs mirror. Edits happen locally; deploys use `/deploy-to-vm` skill.

| Role | Path | Status | Notes |
|---|---|---|---|
| **Local working copy (primary)** | `G:\_OneDrive\OneDrive\Desktop\Py Apps\clawtrol` | **WRITE HERE** | Full clone of `wolverin0/clawtrol`, same remote as VM. Work locally, deploy via `/deploy-to-vm`. |
| VM deployment target | `/home/ggorbalan/clawdeck` | Pulled-from-git | Running Rails service, pulls from origin. Do NOT edit directly unless debugging on the VM is required. |
| VM worktree (alternate branch) | `/home/ggorbalan/factory-workspaces/clawtrol-minimax` | Secondary working copy | Git worktree of `~/clawdeck/.git`. Separate branch, separate working dir. |
| Audit workspace (archived) | `G:\_OneDrive\OneDrive\Desktop\Py Apps\clawtrol-workspace` | Preserved audit docs | AUDIT.md, FIX_PROMPTS.md, FEATURE_IDEAS.md, roadmap.md, obsidian-vault/, screenshots. Moved aside from clawtrol/ during the 2026-04-19 re-clone. Reference only. |
| Scratch patch dump | `G:\_OneDrive\OneDrive\Desktop\Py Apps\clawdeck_remote_work` | Non-canonical | Isolated files from prior remote-work flow, not the live repo |
| Repo comparison clones (Windows) | `G:\_OneDrive\OneDrive\Desktop\Py Apps\gitclones` | Reference only | For benchmarking patterns |
| Repo comparison clones (Ubuntu) | `/home/ggorbalan/gitclone` | Reference only | For benchmarking patterns |
| OpenClaw workspace | `/home/ggorbalan/.openclaw/workspace` | Separate project | Cognitive/memory files, not ClawTrol app code |

### Runtime endpoints

- ClawTrol app URL: `http://192.168.100.186:4001`
- Service unit: `systemctl --user status clawdeck-web.service` (or via `docker compose ps` in `~/clawdeck/`)

### Deploy

Use the `/deploy-to-vm` skill at `.claude/skills/deploy-to-vm/SKILL.md`. Do NOT SSH-edit on the VM for non-emergency changes. The skill enforces clean-local + up-to-date-with-origin pre-flight + refuses to force-push.

## 2) Mandatory Preflight Before Any Edit

All edits happen in the **local clone** at `G:\_OneDrive\OneDrive\Desktop\Py Apps\clawtrol\`. Run these checks locally in order before coding:

```bash
# 1) Confirm cwd is the local clawtrol clone
pwd && git rev-parse --show-toplevel
# Expected: .../clawtrol  (NOT clawtrol-workspace, NOT clawdeck_remote_work)

# 2) Confirm origin matches the canonical repo
git remote get-url origin
# Expected: https://github.com/wolverin0/clawtrol.git

# 3) Confirm branch + dirty state + sync with origin
git branch --show-current
git status --short
git fetch origin && git status -uno   # see if behind/ahead

# 4) Confirm target file exists locally
ls -la <target_path>
```

If `pwd` does not end in `clawtrol` OR origin is not `wolverin0/clawtrol`, stop and re-route. Do NOT SSH to the VM to edit. Pre-deploy verification of the VM state is handled by the `/deploy-to-vm` skill, not the edit preflight.

## 3) Documentation Source-of-Truth Policy

- Canonical roadmap lives in: `docs/roadmaps/`
- Execution evidence lives in: `docs/reports/` and `docs/artifacts/`
- Session handoff context lives in: `handoff.md`

When roadmaps conflict:
1. Keep one canonical roadmap marked ACTIVE.
2. Mark older files as superseded, do not delete historical context.
3. Update checkbox status immediately after each completed step.

## 4) Project Overview

ClawTrol (formerly ClawDeck) is a Rails 8.1 mission control dashboard for AI agents.

Core capabilities:
- Task queue with board-based workflow
- Agent orchestration state (sessions, model routing, validation)
- Swarm idea launcher
- Factory loop automation
- ZeroBitch fleet controls
- Nightshift/cron orchestration

## 5) Architecture Snapshot

### Stack
- Ruby 3.3.x / Rails 8.1
- PostgreSQL (primary/cache/queue/cable)
- Solid Queue, Solid Cache, Solid Cable
- Hotwire (Turbo + Stimulus) + Tailwind
- Puma + Nginx deployment

### P0 Data Contract (Feb 2026)

**`tasks.description` is the HUMAN BRIEF only. Agent output goes to TaskRun.**

Key columns:
- `tasks.description` — Human task brief (never mutated by agents)
- `tasks.original_description` — Backup of original brief
- `tasks.execution_prompt` — Prompt for agent execution (was `execution_plan`)
- `tasks.compiled_prompt` — Pipeline-generated prompt from ERB templates
- `task_runs.agent_output` — Agent findings/output per run
- `task_runs.prompt_used` — Immutable snapshot of the prompt sent to agent
- `task_runs.agent_activity_md` — Markdown transcript summary
- `task_runs.follow_up_prompt` — Follow-up instructions for requeue

Prompt precedence chain: `compiled_prompt || execution_prompt || original_description || description || name`

**Reading agent output:**
```ruby
task.agent_output_text     # reads TaskRun first, falls back to description regex
task.has_agent_output?     # checks TaskRun OR description pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wolverin0/clawtrol](https://github.com/wolverin0/clawtrol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
