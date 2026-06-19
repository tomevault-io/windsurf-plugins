---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

BMAD extension module that integrates sprint tracking with GitLab/GitHub Issues. It's not a runnable application — it's a set of TOML overrides and skills deployed into consuming BMAD projects via the BMAD installer (`npx bmad-method install`).

Requires BMM 6.4.0+ (uniform customize.toml support across all BMM workflows).

## Architecture

Two concepts that must stay aligned:

- **Standalone skill** (`skills/bmad-bmm-issue-sync/SKILL.md`) — the user-facing slash command (`/bmad-bmm-issue-sync`), delegates to `issue-sync/prepare.yaml` + `issue-sync/sync.yaml`
- **Deployed copy** — during setup, this file is copied to `_bmad/_config/custom/bmad-bmm-issue-sync.md` in the consuming project. TOML `on_complete` hooks reference this deployed path.

The standalone skill IS the source. If you edit it, the deployed copy in consuming projects won't update automatically — users must re-run `/bmad-issue-tracking-setup`.

### Issue sync workflow split

The sync task is split into two phases so callers can skip redundant setup:

- **`issue-sync/prepare.yaml`** (steps 1-3) — platform detection, labels, board, PRD issue creation
- **`issue-sync/sync.yaml`** (steps 4-6) — sync issues, mark MR ready, summary (includes its own `check-config` + `find-prd` since context may be compacted)

Callers:
- `sprint-planning/complete.yaml` → `INCLUDE: issue-sync/sync` (steps 4-6 only, prepare ran during sprint planning)
- `sprint-status/complete.yaml` → `INCLUDE: issue-sync/sync` (steps 4-6 only, prepare ran during sprint status)
- `/bmad-bmm-issue-sync` standalone → `INCLUDE: issue-sync/prepare` then `INCLUDE: issue-sync/sync`

## TOML override semantics

Files in `skills/bmad-issue-tracking-setup/assets/custom/` are TOML overrides for BMM workflows:
- `[workflow] activation_steps_append` — array, appends to BMM's activation steps
- `[workflow] on_complete` — scalar, replaces BMM's completion block entirely

All overrides are pure pointers — they reference workflow YAML files that handle the actual logic. The config guard (`common/check-config.yaml` validating `issue_tracking.platform`, `issue_tracking.branch_patterns`, etc.) runs inside each workflow YAML, not in the TOML.

## Key variable conventions in instructions

TOML instructions reference these placeholders — they are NOT config variables, they're resolved at runtime by the AI agent:
- `{prd_key}` — from PRD frontmatter, e.g. `mobile-oidc`
- `{story_key}` — sprint-status entry key, e.g. `1-3-login-form`
- `{epic_num}`, `{story_num}` — extracted from `story_key` (first two dash-separated numbers)
- `{prd_branch}` — `branch_patterns.prd` resolved with `{prd_key}`, e.g. `feat/mobile-oidc/prd`
- `{story_branch}` — `branch_patterns.story` resolved with `{prd_key}` and `{story_key}`
- `{sep}` — `::` for GitLab, `:` for GitHub (label separator)
- `$MR_HOST`, `$MR_PROJECT` — git remote host/project for MR operations (GitLab); same as `$HOST`/`$PROJECT_PATH` when platforms match
- `$MR_OWNER`, `$MR_REPO` — git remote owner/repo for PR operations (GitHub); same as `$OWNER`/`$REPO` when platforms match

## Issue title formats

All workflows that create issues use these title formats. They must stay consistent — `create-issue.yaml` searches by title to avoid duplicates.

| Type | Format | Set by |
|------|--------|--------|
| PRD | `PRD: {prd_key}` | `bmad-prd/complete.yaml`, `create-prd/complete.yaml`, `issue-sync/prepare.yaml` |
| Story | `Story {epic_num}.{story_num}: {title}` | `create-story/complete.yaml`, `sync-issues.yaml` |
| Epic | `Epic {n}: {title}` | `sync-issues.yaml` |
| Retrospective | `Retrospective: Epic {n}` | `retrospective/complete.yaml` |

For stories, `{title}` is extracted from the story file heading (`# Story 1.4: Login Form` → `Login Form`). During initial sync (sprint-planning), story files don't exist yet — the title is derived from the entry key (`1-4-login-form` → `Login Form`). Both paths produce the same format.

## Branch/MR flow

Branch setup happens in activation (before BMM workflow runs). The BMM workflow creates files directly in the worktree. on_complete handles commit/push/issue/MR. Never commit on PRD for story work.

| Workflow | Activation | on_complete | MR direction |
|----------|-----------|-------------|--------------|
| bmad-prd (6.8.0+) | Detect intent: create → ask key + create worktree; update/validate → find worktree | Create → issue + commit + push + draft MR; update → update description | PRD → default (draft, create only) |
| create-prd (6.4.0–6.7.x) | Create/switch to PRD worktree | Commit + push + issue + draft MR | PRD → default (draft) |
| create-architecture | Switch to PRD worktree | Commit + push | (PRD worktree) |
| create-ux-design | Switch to PRD worktree | Commit + push | (PRD worktree) |
| create-epics-and-stories | Switch to PRD worktree | Commit + push | (PRD worktree) |
| sprint-planning | Switch to PRD worktree | Trigger issue sync (steps 4-6) | (PRD worktree) |
| edit-prd (6.4.0–6.7.x) | Switch to PRD worktree | Update PRD issue description | (PRD worktree) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jrevillard/bmad-issue-tracking](https://github.com/jrevillard/bmad-issue-tracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
