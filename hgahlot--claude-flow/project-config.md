---
trigger: always_on
description: <!-- FILL THIS IN: Describe your project in 2-3 sentences, tech stack, key constraints -->
---

# Claude Code Configuration

## Project Overview
<!-- FILL THIS IN: Describe your project in 2-3 sentences, tech stack, key constraints -->

State files:
- `PROJECT.md` — project overview, goals, tech stack, constraints
- `REQUIREMENTS.md` — functional and non-functional requirements
- `ROADMAP.md` — milestone breakdown with status
- `STATE.md` — current session state, last completed task, next task queue

---

## Installed Tools & Commands

### GSD (Get Shit Done)
Spec-driven development lifecycle manager.

| Command | Purpose |
|---|---|
| `/gsd:new-project` | Initialize a new project with deep context gathering |
| `/gsd:discuss-phase` | Gather phase context through adaptive questioning |
| `/gsd:plan-phase` | Create detailed PLAN.md with verification loop |
| `/gsd:execute-phase` | Execute all plans with wave-based parallelization |
| `/gsd:verify-work` | Validate built features through conversational UAT |
| `/gsd:ship` | Create PR, run review, and prepare for merge |
| `/gsd:pause-work` | Create context handoff when pausing mid-phase |
| `/gsd:resume-work` | Resume work from previous session with full context |
| `/gsd:quick` | Execute a quick task with GSD guarantees |
| `/gsd:fast` | Execute a trivial task inline (no planning overhead) |
| `/gsd:autonomous` | Run all remaining phases autonomously |
| `/gsd:health` | Diagnose planning directory health |
| `/gsd:checkpoint` | Save current state to STATE.md |
| `/gsd:ui-phase` | Generate UI design contract (UI-SPEC.md) |
| `/gsd:ui-review` | 6-pillar visual audit of implemented frontend code |
| `/gsd:debug` | Systematic debugging with persistent state |
| `/gsd:session-report` | Generate session report with token usage |
| `/gsd:stats` | Display project statistics |

### gstack (by Garry Tan, YC CEO) — GLOBAL
Located at `~/.claude/skills/gstack/`. 25 skills covering the full deployment pipeline.

Key commands:
| Command | Purpose |
|---|---|
| `/office-hours` | YC-style product brainstorming — 6 forcing questions + competitor research + design doc |
| `/autoplan` | Runs CEO + eng + design reviews of a plan sequentially without prompts |
| `/plan-ceo-review` | Founder scope review (EXPANSION/HOLD/REDUCE), 10-star product exercise |
| `/plan-eng-review` | Eng manager review with ASCII test coverage diagrams + blast radius analysis |
| `/plan-design-review` | Designer's eye plan review — interactive, 0-10 scoring |
| `/review` | Superior PR review: CRITICAL+INFORMATIONAL, Greptile integration, adversarial |
| `/qa` | Full QA with headless browser, 10-phase process, fix loop (up to 50 fixes) |
| `/ship` | Full release pipeline: tests + coverage + review + CHANGELOG + bisectable commits + PR |
| `/land-and-deploy` | Merge + CI wait + deploy |
| `/canary` | Post-deploy monitoring loop |
| `/benchmark` | Performance regression: TTFB, FCP, LCP, bundle sizes |
| `/investigate` | Systematic root-cause debugging (Iron Law: no fixes without root cause) |
| `/retro` | Weekly retrospective with real git metrics |
| `/careful` | PreToolUse hook blocking dangerous commands |
| `/design-consultation` | Build complete DESIGN.md + HTML preview from scratch |
| `/design-review` | Live site visual audit via screenshots |

### Compound Engineering (by Every) — GLOBAL PLUGIN
Installed via plugin marketplace. AI skills and agents that make each unit of engineering work easier than the last. Philosophy: 80% planning and review, 20% execution. Each cycle compounds.

**Core workflow:** Brainstorm → Plan → Work → Review → Compound

| Command | Purpose |
|---|---|
| `/ce:ideate` | Discover high-impact project improvements through divergent ideation |
| `/ce:brainstorm` | Explore requirements and approaches before planning |
| `/ce:plan` | Turn ideas into detailed implementation plans with repo-aware research |
| `/ce:work` | Execute plans with worktrees and task tracking |
| `/ce:review` | Multi-agent tiered code review with confidence-gated findings |
| `/ce:compound` | Document learnings to make future work easier |
| `/ce:compound-refresh` | Refresh stale learnings against current codebase |

**Additional skills:**

| Command | Purpose |
|---|---|
| `/agent-browser` | Browser automation for AI agents |
| `/frontend-design` | Build web interfaces with genuine design quality |
| `/onboarding` | Generate ONBOARDING.md for new contributors |
| `/coding-tutor` | Personalized coding tutorials using your codebase |
| `/git-commit` | Clear, value-communicating commit messages |
| `/git-commit-push-pr` | Commit → push → open PR in one step |
| `/git-worktree` | Manage git worktrees for parallel development |
| `/git-clean-gone-branches` | Clean up local branches with gone remotes |
| `/lfg` | Full autonomous engineering workflow |
| `/slfg` | Full autonomous workflow using swarm mode |
| `/reproduce-bug` | Systematically reproduce a bug from a GitHub issue |
| `/resolve-pr-feedback` | Resolve PR review comments in parallel |
| `/document-review` | Review docs using parallel persona agents |
| `/claude-permissions-optimizer` | Optimize Claude Code permission allowlists |
| `/orchestrating-swarms` | Multi-agent swarm coordination |
| `/todo-create` | Create durable work items in file-based todo system |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hgahlot/claude-flow](https://github.com/hgahlot/claude-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
