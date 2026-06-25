---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (`agent-flow`) that automates bug-fix workflows, feature implementation, and project scaffolding. It provides specialized agents that are orchestrated by commands to take an issue from triage through fix, review, test, and publish — or to scaffold a new project from scratch. The plugin is generic — all project-specific configuration lives in the consuming project's CLAUDE.md under `## Automation Config`.

**Author:** Filip Sabacky
**Installation:** `claude plugin marketplace add <path-to-repo>`, then `claude plugin install agent-flow@agent-flow`

## Repository Structure

No build system, no dependencies. Manual test suite in `tests/`. This is a pure plugin of markdown definitions.

- `.claude-plugin/` — Plugin metadata (`plugin.json`, `marketplace.json`)
- `agents/` — 17 agent definitions (markdown with YAML frontmatter)
- `skills/` — 17 skills (slash commands)
- `docs/guides/` — Installation and configuration guides
- `docs/reference/` — Command, agent, pipeline, and config reference
- `examples/` — Config templates, custom agent examples, MCP config examples
- `checklists/` — Pipeline phase checklists (review, test, publish)
- `tests/` — Test harness with scenarios and CI workflow
- `.agent-flow/` — Per-run pipeline state files (state.json, pipeline.log, browser artifacts)
- `state/` — State schema documentation
- `core/` — 17 shared pipeline pattern contracts

## Architecture: 2-Layer System

**Skills** (orchestration — WHAT to do): `/analyze-bug`, `/autopilot`, `/changelog`, `/check-setup`, `/create-backlog`, `/discuss`, `/fix-bugs`, `/implement-feature`, `/metrics`, `/onboard`, `/prioritize`, `/publish`, `/scaffold`, `/setup-agents`, `/setup-mcp`, `/sprint-plan`, `/version-check`
**Agents** (specialists — HOW to do it): acceptance-gate, analyst, architect, backlog-creator, browser-agent, deployment-verifier, fixer, priority-engine, publisher, reviewer, rollback-agent, scaffolder, spec-analyst, spec-reviewer, spec-writer, sprint-planner, test-engineer

Skills read `## Automation Config` from the project's CLAUDE.md and dispatch agents. Skills contain zero project-specific logic.

## Bug-Fix Pipeline

```
Issue tracker query → ANALYST --phase triage (sonnet, +AC extraction, +complexity, +reproduction_steps)
  → ANALYST --phase impact (sonnet) → [BROWSER-AGENT --phase reproduce (sonnet, optional)] → [Pre-fix hook]
  → FIXER ↔ REVIEWER (opus, +AC fulfillment check)
  → [Post-fix hook + custom agent] → [Smoke check (build + test)] → TEST-ENGINEER (sonnet)
  → [TEST-ENGINEER --e2e (sonnet, optional)] → [BROWSER-AGENT --phase verify (sonnet, optional)] → [Acceptance gate (conditional: AC ≥ 3 or complexity ≥ M)]
  → [Pre-publish hook + custom agent] → PUBLISHER (haiku)
```

Retry limits are configurable via `Retry Limits` section in Automation Config (defaults: 5 fixer iterations, 3 test attempts, 3 build retries, 5 spec iterations, 3 root cause iterations).

Each agent can **Block** the issue (set state, add comment using Block Comment Template, move on). On block from fixer/reviewer/test-engineer: **rollback-agent** reverts git state. Hooks and custom agents can be inserted at 4 points (see `skills/fix-bugs/SKILL.md` for full pipeline).

## Feature Pipeline

```
Issue tracker query → SPEC-ANALYST (sonnet, +AC writeback)
  → ARCHITECT (opus, +maps_to traceability)
  → [AC coverage check] → [Decomposition decision] → [Create tracker subtasks]
  → FIXER ↔ REVIEWER (opus, +AC fulfillment check)
  → [Smoke check (build + test)] → TEST-ENGINEER (sonnet) → [Acceptance gate (always in decomposition, skipped in single-pass)]
  → PUBLISHER (haiku)
```

## Scaffold Pipeline

```
[0-INFRA: infra declaration] → [0-MCP: MCP check]
  → User description → SPEC-WRITER ↔ SPEC-REVIEWER (opus)
  → [Spec checkpoint] → SCAFFOLDER (sonnet, +test infrastructure, +scorecard)
  → Validate → Git init → [4d: push] → [4e: tracker issues]
  → ARCHITECT (opus, +maps_to) → [Feature plan checkpoint]
  → FIXER ↔ REVIEWER (opus) → TEST-ENGINEER (sonnet)
  → [Spec compliance check (spec-reviewer --verify)]
  → TEST-ENGINEER --e2e (sonnet) → Final report
```

Pipeline mode is selected via a flag at invocation time:
- **default** — human-in-the-loop with review checkpoints after each major phase
- **`--yolo`** — automated run, all checkpoints skipped, pipeline runs to completion without pausing
- **`--step-mode`** — pause before every individual agent step for fine-grained control

With `--no-implement`: `[0-INFRA] → [0-MCP] → STACK-SELECTOR (sonnet) → SCAFFOLDER (sonnet) → Validate → Git init → [push if SC ready]` (skeleton only, no implementation).

In spec-first mode, the specification is saved as a `spec/` folder in the project root (spec/README.md, spec/architecture.md, spec/verification.md, spec/epics/*.md). This folder is the single source of truth for all downstream agents.

## Agent Definition Format

Every agent file in `agents/` follows this exact structure:

```markdown
---
name: agent-name
description: One-line description used by Claude Code's Task tool
model: sonnet | opus | haiku
style: Short communication style descriptor
---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asysta-act/agent-flow](https://github.com/asysta-act/agent-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
