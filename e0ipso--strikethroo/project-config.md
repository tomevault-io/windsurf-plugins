---
trigger: always_on
description: Primary context source for AI-assisted work in this repository.
---

# AGENTS.md

Primary context source for AI-assisted work in this repository.

## Quick Start

```bash
# Build, then run init or serve
npm run build
npm start init --harnesses claude --destination-directory /path/to/project   # --force to overwrite all
node dist/cli.js serve                                                       # or: npx strikethroo serve

# Development
npm run dev           # Watch mode compilation
npm test              # Full gate: unit (Vitest) then e2e (@playwright/test)
npm run lint:fix      # Auto-fix style
```

`init` bootstraps the `.ai/strikethroo/` workspace (and copies Claude agents); it does **not** install skills. It uses SHA-256 hash tracking in `.ai/strikethroo/.init-metadata.json` to detect and protect user-modified files; `--force` bypasses the prompts for automation.

The workflow itself ships as **Agent Skills** (harness-agnostic — one `SKILL.md` works on any harness supporting the format). Install once with `npx skills add e0ipso/strikethroo` (append `@<tag>` to pin); the matching skill auto-loads on intent.

---

## Glossary

- **Work order** — The user's request describing what they want accomplished.
- **Plan** — Comprehensive document covering requirements, architecture, risks, and success criteria.
- **Execution blueprint** — All tasks organized into dependency-mapped phases. Output of task generation.
- **Phase** — A group of tasks that execute in parallel. Phases run in sequence.
- **Task** — An atomic unit of work with 1-2 skills and clear acceptance criteria. Executed by a sub-agent.
- **Sub-agent** — A specialized AI agent executing a single task with focused, clean context.

---

## Project Overview

This CLI tool initializes AI-assisted development environments with hierarchical task management. It transforms complex programming requests into atomic, validated implementations through staged refinement — managing AI context load, enforcing YAGNI scope control, and ensuring working code through integrity-focused testing.

---

## Strikethroo Plan and Task Management System

### Workflow Skills

Each step is an Agent Skill that auto-loads when the user's request matches its description:

- `st-create-plan` — strategic plan creation with mandatory clarification gates (prevents assumption-based planning).
- `st-generate-tasks` — task decomposition into dependency-mapped atomic units (enforces 20-30% task reduction, 1-2 skills max per task).
- `st-execute-blueprint` — execution orchestration across all tasks, with dependency-aware parallelism and quality gates.
- `st-refine-plan` — plan refinement loop: a second assistant "red teams" an existing plan, asks questions, and applies refinements. Bridges plan creation and task generation.
- `st-execute-task` — single-task execution.
- `st-full-workflow` — end-to-end chaining of plan → tasks → blueprint for hands-off runs.

### Key Design Principles

- **Atomic decomposition** — max 2 skills per task (3+ triggers subdivision); automatic skill inference; explicit dependency mapping.
- **Scope control (YAGNI)** — anti-pattern enumeration, "Is this explicitly mentioned?" validation, 20-30% minimization targets, every task traceable to an explicit requirement.
- **Test philosophy: "Write a few tests, mostly integration"** — selective meaningful coverage over completeness; real filesystem operations over mocking; focus on custom logic, critical workflows, and edge cases; don't test third-party/framework features, trivial getters/setters, or obvious CRUD.

---

## Skills Layer

Skills live under `templates/harness/skills/<name>/` (no top-level `skills/` dir; flat, no nesting). Each skill's `SKILL.md` and its compiled `.cjs` bundle under `scripts/` are assembled/bundled at build time — source and output share the same per-skill tree.

The six shipping skills are the workflow skills listed above (`st-create-plan`, `st-generate-tasks`, `st-execute-blueprint`, `st-refine-plan`, `st-execute-task`, `st-full-workflow`).

### TypeScript source of truth

Runtime logic each skill needs is authored once in TypeScript under `src/skill-scripts/`, with shared helpers (frontmatter parsing, plan/archive scanning, root discovery) in `src/skill-scripts/shared/`. The subtree type-checks via `tsconfig.skill-scripts.json` and lints with `src/`, but its output is produced by the bundler — the main `tsconfig.json` excludes `src/skill-scripts/**` from emit so `dist/` stays the CLI's domain.

### Prompt source of truth

Each `SKILL.md` is assembled at build time from source templates in `src/skill-prompts/`. Shared procedural blocks live in `src/skill-prompts/sections/`, referenced via `{{include sections/<name>.md}}`; per-skill differences use `{{variable}}` substitution from the template's frontmatter `vars` block. See `src/skill-prompts/README.md` for assembly mechanics and `src/skill-prompts/AUTHORING.md` for the prompt-authoring house style (form-over-narrative, "no nuance clauses", anti-rationalization tables, Skill Discovery Optimization for descriptions, imperative phrasing) — read it before editing prompt content.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [e0ipso/strikethroo](https://github.com/e0ipso/strikethroo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
