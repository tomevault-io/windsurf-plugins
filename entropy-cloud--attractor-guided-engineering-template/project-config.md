---
trigger: always_on
description: `<project-name>` uses a lightweight Attractor-Guided Engineering workflow for AI-assisted application development.
---

# AGENTS.md

## Project Intent

`<project-name>` uses a lightweight Attractor-Guided Engineering workflow for AI-assisted application development.

This repository is for an application-layer product, not a framework-core project.

The repo is the source of truth. Chat is only a temporary working surface.

Before writing non-trivial code, agents must first understand:

- `docs/context/project-context.md`
- `docs/context/ai-autonomy-policy.md`
- `docs/context/codebase-map.md`
- the active requirement listed in project context
- the active owner doc listed in project context
- the relevant raw inputs under `docs/input/` when requirement meaning depends on source material

Read `docs/context/source-of-truth-and-precedence.md` when facts conflict or you are unsure which artifact owns the answer.
Read `docs/process/application-development-workflow.md` when planning or workflow decisions are part of the task.

## Task Routing

Before writing code, agents MUST classify the task first:

1. Determine the task type:
   - requirement clarification
   - app-layer design change
   - architecture change
   - implementation-only change
   - bug investigation
   - verification or audit work
2. Use `docs/index.md` to read the owner docs for that task type before acting.
3. Check `docs/skills/README.md` for candidate reusable skills before drafting or revising a plan.
4. For non-trivial work, record the chosen route and planned skill usage in the plan before implementation.

Do not jump from a feature request directly to code unless the route is already obvious from the active requirement and owner docs.

## Operating Rules

1. Prefer file-in, file-out collaboration.
2. Do not treat chat summaries as durable project memory.
3. Do not jump from raw PM text or prototype screenshots straight to code when scope is still unclear.
4. If input is ambiguous, first create or update a file in `docs/discussions/` or `docs/requirements/`.
5. Create or update a plan before implementation when the planning triggers below apply.
6. Keep `docs/design/` and `docs/architecture/` focused on the current supported baseline, not migration history.
7. Keep logs short, dated, and append-only.
8. Record non-obvious regressions in `docs/bugs/`.
9. If prototype and implementation diverge materially, capture the reason in `docs/retrospectives/` instead of silently moving on.
10. Promote repeated process lessons into `docs/skills/` or `docs/audits/` only when the pattern is recurring enough to justify reuse.
11. For high-risk or high-ambiguity requirement, design, or plan drafts, request an independent subagent or reviewer pass and revise until major objections are resolved. Every created plan MUST pass an independent plan audit before implementation begins and an independent closure audit before being marked complete.
12. Keep code comments minimal. Prefer self-explanatory code; add only rare comments when a local constraint is otherwise easy to misread.
13. When a referenced file is not found at its expected path, check `docs/archive/` before concluding it does not exist. Archived files retain their original relative name under `docs/archive/`. Do not move files to `docs/archive/` without human approval.
14. Treat reusable skills as method selectors, not substitutes for requirements, design, or architecture docs. Business knowledge belongs in owner docs first.
15. When the same error pattern keeps recurring, do not stop at prose-only lessons. First promote it into a reusable audit prompt, checklist, or review playbook when that method is still missing. If the defect pattern still recurs, then evaluate promotion into a heuristic script, static check, lint rule, CI guard, or codemod, tuned to the copied project's real conventions and false-positive tolerance.

## Read This First

- `docs/context/project-context.md`
- `docs/context/ai-autonomy-policy.md`
- `docs/context/codebase-map.md`
- the active requirement listed in `docs/context/project-context.md`
- the active owner doc listed in `docs/context/project-context.md`

Read additionally when needed:

- `docs/context/source-of-truth-and-precedence.md` for ownership or conflict questions
- `docs/context/conventions.md` for project-wide conventions
- `docs/process/application-development-workflow.md` for workflow questions
- `docs/index.md` when you need routing beyond the active files

## Documentation Ownership

- `docs/context/` owns mandatory AI context, source-of-truth precedence, and project-wide conventions.
- `docs/backlog/` owns prioritized candidate work and AI-ready next actions.
- `docs/input/` owns raw external inputs such as PM notes, card docs, article extracts, prototype references, and copied source material.
- `docs/discussions/` owns requirement clarification conversations and unresolved question records.
- `docs/requirements/` owns implementation-ready requirement synthesis.
- `docs/design/` owns stable app-layer business and feature design.
- `docs/architecture/` owns cross-cutting technical and module-boundary truth.
- `docs/lessons/` owns durable reusable lessons extracted from bugs, audits, and retrospectives.
- `docs/plans/` owns execution and closure criteria for non-trivial work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entropy-cloud/attractor-guided-engineering-template](https://github.com/entropy-cloud/attractor-guided-engineering-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
