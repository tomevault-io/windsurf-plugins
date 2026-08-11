---
trigger: always_on
description: > **This file is a table of contents, not an encyclopedia.**
---

# AGENTS.md: Project Router

> **This file is a table of contents, not an encyclopedia.**
> Keep it under ~150 lines. Point agents to references. Do not embed content here.
> If guidance needs to exist, put it in `docs/` and link from here.

## What this is

Team is a Claude Code plugin that orchestrates specialized agents to implement features end-to-end. The orchestrator (the main Claude Code session) walks a linear phase table, and persists state as artifact files in `docs/plans/<id>/`. That per-id directory carries YAML frontmatter with phase and revision metadata. The orchestrator coordinates live progress through TodoWrite. See [docs/architecture.md](docs/architecture.md) for the full design.

> **North star: read [docs/vision.md](docs/vision.md) and [docs/ethos.md](docs/ethos.md).** Team is a *loop-driven development system*: a human fills the Backlog and reviews finished work. Everything in between (groom → start → implement → open PR) runs autonomously. The ethos explains *why* the autonomous middle can be trusted. Every agent should understand this end state, which is the target the whole project moves toward.

## Runtime vs. development

This project produces a **distributed plugin**. Two contexts exist:

**Runtime** (`agents/`, `skills/`, `hooks/`, and the host manifest dirs `.claude-plugin/`, `.codex-plugin/`, `.agents/plugins/`) ships to end users. Fires when someone installs the Team plugin and runs `/team`. Changes here affect all users.

**Development** (`.claude/`) is our workspace tooling. Fires only when developing the plugin itself. Never distributed.

| Concern | Where it lives | Who runs it |
|---------|---------------|-------------|
| Pipeline agents, skills, hooks | `agents/`, `skills/`, `hooks/` | End users |
| Plugin manifests | `.claude-plugin/` (Claude Code), `.codex-plugin/` + `.agents/plugins/` (Codex) | End users |
| Registry sync validation | `.claude/hooks/check-registry-sync.mjs` | Plugin developers |
| Pre-merge version gate | `.claude/hooks/pre-merge-guard.mjs` | Plugin developers |
| Dev acceptance scripts | `.claude/scripts/` | Plugin developers |
| Dev settings/hooks | `.claude/settings.json` | Plugin developers |
| Work tracking | [GitHub Project board](https://github.com/users/bostonaholic/projects/5/views/1) | Plugin developers |
| Behavioral regression harness | `tests/`, `evals/` | Plugin developers |
| Versioning & release automation | [docs/versioning.md](docs/versioning.md), `.claude/skills/version-bump/`, `.claude/scripts/next-version.sh`, `.github/workflows/` | Plugin developers |
| Dev install, per harness | `script/dev-install`/`dev-uninstall` (dispatch), `dev-install-<harness>` | Plugin developers |

**Rule of thumb:** If it validates that the plugin is *built correctly*, it is a dev concern (`.claude/`). If it runs *as part of the plugin's functionality*, it is runtime (`hooks/`).

## Design philosophy

Agents are **decoupled microservices**. Each consumes a predecessor artifact on disk, does work, and writes its output artifact to `docs/plans/` (with YAML frontmatter on every artifact). The orchestrator walks a linear phase table in `skills/team/SKILL.md`. `skills/team/registry.json` lists the 13 agents as a phase-tagged inventory.

## Pipeline

```
WORKTREE → QUESTION → RESEARCH → DESIGN → STRUCTURE → PLAN → IMPLEMENT → PR
```

Team runs **QRSPI** (Worktree-Question-Research-Design-Structure-Plan-Implement-PR). There are **no mid-run human gates**. An adversarial design review gates the Design (~200-line alignment doc), and the orchestrator records the verdicts to `design-review-<n>.md`. The human's checkpoint is the PR review at the end. The Structure (~2-page vertical-slice breakdown) is produced autonomously and advances to Plan with no approval wait. Research is **isolated**: the researcher reads only `questions.md`, never `task.md` or the user's framing. The Plan is a tactical artifact for the implementer, not for human review. Implement is a sub-pipeline (test-first → slice execution → 5-reviewer adversarial verify with hard-gate retry loop). The whole run is autonomous with mechanical gates.

## Entry points

| Command | Phase |
|---------|-------|
| `/team <desc>` | Full 8-phase QRSPI pipeline |
| `/team-fix <bug>` | Compressed bug-fix pipeline (no QRSPI ceremony) |
| `/team-worktree` | Leading WORKTREE phase: create the home worktree. In a full run it is automatic and first. Standalone, it consumes `plan.md` post-PLAN for manual recovery or multi-repo setup |
| `/team-question <desc>` | Decompose intent into task + questions + brief |
| `/team-research` | Isolated codebase research (runs Question if missing) |
| `/team-design` | Draft the design. An adversarial design review gates advancement |
| `/eng-design-doc-review` | Adversarial fresh-context audit of `design.md`. Its Review brief doubles as the pipeline's design-review gate, and standalone use remains |
| `/team-structure` | Break design into vertical slices (autonomous) |
| `/team-plan` | Tactical plan from the structure |
| `/team-implement` | Test-first + slice execution + 5-reviewer verify |
| `/team-pr` | Commit + open PR |

## Agents (13)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bostonaholic/team](https://github.com/bostonaholic/team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
