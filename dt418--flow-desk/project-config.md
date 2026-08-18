---
trigger: always_on
description: This repository is designed for long-running coding-agent work. The goal is not to maximize raw code output. The goal is to leave the repo in a state where the next session can continue without guessing.
---

# FlowDesk — Agent Instructions

This repository is designed for long-running coding-agent work. The goal is not to maximize raw code output. The goal is to leave the repo in a state where the next session can continue without guessing.

## Tech Stack

| Layer      | Technology                                                 |
| ---------- | ---------------------------------------------------------- |
| Frontend   | React 18 + Vite + TypeScript + Tailwind CSS v4 + shadcn/ui |
| Backend    | Hono + Node.js + TypeScript                                |
| Database   | PostgreSQL 16                                              |
| Cache      | Redis 7                                                    |
| ORM        | Prisma                                                     |
| Realtime   | Socket.IO (Redis adapter)                                  |
| Auth       | JWT in httpOnly cookie (bcrypt + Google OAuth)             |
| AI         | OpenAI-compatible (custom baseUrl + model)                 |
| Deployment | Docker Compose (self-hosted)                               |

## Startup Workflow

Before writing code:

1. Confirm the working directory with `pwd` (expect `/home/thanh/flow-desk`).
2. Read `claude-progress.md` for the latest verified state and next step.
3. Read `feature_list.json` and choose the highest-priority unfinished feature.
4. Review recent commits with `git log --oneline -5`.
5. Run `./init.sh`.
6. Run the required smoke or end-to-end verification before starting new work.

If baseline verification is already failing, fix that first. Do not stack new feature work on top of a broken starting state.

## Feature workflow (`plan-feature`)

When planning or shipping a **product feature** (new capability, ROADMAP item, or work that updates `feature_list.json`):

1. **Invoke** the `plan-feature` skill before writing feature code.
2. Canonical path: `.agents/skills/plan-feature/SKILL.md` (Agent Skills standard).
3. Slash: `/plan-feature <request | ROADMAP id | next>` (Claude, Pi, OpenCode, Cursor, Grok).

The skill **orchestrates** Superpowers (`brainstorming` → `writing-plans` → `subagent-driven-development` or `executing-plans` → `verification-before-completion` → `finishing-a-development-branch`) plus FlowDesk harness rules (single active feature, schema hygiene, `pnpm verify` evidence). Do not re-implement that pipeline ad hoc in chat.

**Multi-agent (one pattern for all):** every host skill dir is a **directory symlink** to canonical trees under `.agents/skills/`; every slash command is a **file symlink** to `.pi/prompts/*.md`. Rebuild after clone: `pnpm sync:agents` (also run by `./init.sh`). Details: `.agents/skills/plan-feature/references/adapters.md`.

## Harness: FlowDesk agent team

Based on [revfactory/harness](https://github.com/revfactory/harness) (L3 team-architecture factory). Domain agents + skills for multi-role ship/review — **complements** `plan-feature`, does not replace it.

**Goal:** Coordinate explorer → implementer → security+QA fan-out → durable docs with `_workspace/` artifacts.

**Trigger:** multi-role ship/review, "flowdesk team", "ship with harness", `/flowdesk-team` → use skill `flowdesk-team`. Product feature design/plan gates → still `plan-feature`. Reconfigure agents/skills → skill `harness`.

| Kind          | Path                                                            |
| ------------- | --------------------------------------------------------------- |
| Agents        | `.claude/agents/fd-*.md`                                        |
| Orchestrator  | `.agents/skills/flowdesk-team/`                                 |
| Domain skills | `flowdesk-implement`, `flowdesk-security-review`, `flowdesk-qa` |
| Meta-factory  | `.agents/skills/harness/` (upstream workflow + references)      |

**Change log:**

| Date       | Change                                                                                | Target                                                     | Reason                                                                                                                                              |
| ---------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-07-18 | Initial FlowDesk team (5 agents + 4 domain skills + harness meta)                     | `.claude/agents/*`, `.agents/skills/flowdesk-*`, `harness` | Setup from revfactory/harness                                                                                                                       |
| 2026-07-18 | plan-feature v2.2: add missing Superpowers skills to inheritance + execution dispatch | `.claude/skills/plan-feature/SKILL.md`                     | Close orchestration gaps: `dispatching-parallel-agents`, `receiving-code-review`, `using-git-worktrees` now properly wired with activation triggers |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dt418/flow-desk](https://github.com/dt418/flow-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
