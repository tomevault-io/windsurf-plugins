---
trigger: always_on
description: Jig is the AI engineering workflow framework for teams. It guides AI agents through a structured pipeline — from ticket to post-mortem — with quality gates at every stage. Named after the manufacturing tool that holds workpieces and guides tools to produce consistent results.
---

# Jig Framework

Jig is the AI engineering workflow framework for teams. It guides AI agents through a structured pipeline — from ticket to post-mortem — with quality gates at every stage. Named after the manufacturing tool that holds workpieces and guides tools to produce consistent results.

This repo IS Jig. It also uses itself to develop itself (Consumer Zero).

## Core Philosophy

1. **Convention over configuration.** Jig has opinions. Teams override what they need; everything else just works.
2. **The framework is the nervous system, team skills are organs.** Jig discovers, loads, and orchestrates team-created skills. They're part of the system, not separate from it.
3. **Full lifecycle.** Ticket to post-mortem. Every stage has a skill, every handoff has a quality gate.
4. **Language-agnostic core.** The pipeline works for any stack. Stack-specific expertise comes from team skills and starter packs.

## Architecture

Jig has three discovery layers, in priority order:

```
1. team/           <- Highest priority (team overrides)
2. packs/          <- Pack defaults (starter kits)
3. core/           <- Framework defaults
```

If a team skill has the same name as a core skill, the team version wins. This is how teams customize without editing framework files.

### The Pipeline

Every development task flows through stages. `kickoff` orchestrates the full pipeline:

```
DISCOVER → BRAINSTORM → PLAN → EXECUTE → REVIEW → SHIP → LEARN
```

Work type (bug/feature/improvement/task) determines which stages run and at what depth. Configured in `jig.config.md`.

### How Skills Compose

- **Direct invocation** — workflow skills invoke each other by name (`kickoff` → `brainstorm` → `plan`)
- **Concerns checklist** — `jig.config.md` maps engineering concerns to skills. `brainstorm` reads the config and loads relevant skills during design.
- **Specialist dispatch** — `review` discovers specialist `.md` files from all three directories, filters by glob match, and dispatches as parallel subagents.

## Inventory

### Core Skills

Run `find core/skills -name SKILL.md` for the current list; this table summarizes purpose.

| Skill | Purpose |
|-------|---------|
| `kickoff` | Pipeline orchestrator — classifies work, routes through stages |
| `init` | First-run setup — auto-detects environment, interviews, generates config |
| `ticket` | Ticket creation — routes to the configured ticket-system pack (Linear/Jira/GitHub) |
| `brainstorm` | Collaborative design exploration with configurable concerns checklist |
| `prd` | PRD creation with enforceable acceptance checklists (feeds into spec reviewer) |
| `plan` | Spec → implementation plan with bite-sized TDD tasks |
| `build` | Plan execution — analyzes task graph, auto-selects parallel or serial |
| `team-dev` | Parallel agent execution with staggered quality gates (called by `build`) |
| `sdd` | Serial execution with two-stage review per task |
| `review` | Specialist swarm — dispatches parallel reviewers, scores findings |
| `pr-create` | PR creation with voice/tone standards and test plan |
| `pr-respond` | PR comment response — analyze, fix, commit, push, reply, resolve |
| `postmortem` | Post-merge retrospective with specialist/logic reviewer diagnosis |
| `debug` | Systematic debugging — root cause before fixes, always |
| `verify` | Evidence before assertions — run it before claiming it works |
| `tdd` | Red-green-refactor discipline |
| `finish` | Branch completion — merge, PR, keep, or discard |
| `worktree` | Create/remove provisioned worktrees for parallel dev |
| `extend` | Framework extension assistant — scaffolds new skills, specialists, packs |

### Core Agents

| Agent | Purpose |
|-------|---------|
| `commit` | Conventional commits with hook awareness |
| `code-review` | Dispatches review swarm, delivers scored report |
| `pr-review` | Posts inline PR comments with suggestion blocks |

### Core Specialists

Core specialists live in `core/specialists/`. They fall into four groups:

- **Code review** — universal code-quality reviewers dispatched by `review` in code mode (e.g., `security`, `dead-code`, `error-handling`).
- **PRD review** — dispatched by `review` in prd mode against design docs (e.g., `data-dependency`, `ui-conflict`).
- **Plan review** — dispatched by `review` in plan mode against plan docs (e.g., `task-dependency`, `migration-safety`).
- **Cross-stage** — apply across modes (e.g., `blast-radius`, `state-completeness`).

Run `ls core/specialists/` for the current list.

### Packs

Packs live in `packs/`. Each ships a `pack.json` declaring its skills and specialists. Two current kinds:

- **Content packs** (e.g., `engineering`) — ship domain skills and specialists that a team can adopt wholesale.
- **Integration packs** (e.g., `linear`, `jira`, `github-issues`) — teach the core `ticket` skill how to talk to a specific platform, keyed by `ticket-system` in `jig.config.md`.

See each pack's `pack.json` and `README.md` for what it ships.

## Self-Hosting Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duronext/jig](https://github.com/duronext/jig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
