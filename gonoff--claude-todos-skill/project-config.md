---
trigger: always_on
description: Per-project todo tracking with a relational project-knowledge brain — decisions, business rules, contradictions, stakeholders, scope changes — and a local broadsheet dashboard at localhost:5757. Use when the user wants to plan, sync, or close out implementation work in a project, capture a decision/rule/contradiction/feedback to remember later, or browse the dashboard.
---


# Todos Skill

Tracks implementation todos per project AND captures the relational project intelligence around them. Each project gets:

- `.claude/todos.json` — tasks and phases (committed to repo).
- `.claude/brain.db` — SQLite "project brain": stakeholders, feedback, decisions, contradictions, business rules, scope changes, architecture notes, **per-task plans + their planning Q&A** (committed to repo).
- `.claude/{docs,notes,diagrams}/` — long-form markdown and Mermaid diagrams referenced from `brain.db`.

A local Bun server at `http://localhost:5757` reads every tracked project and renders an aggregated newspaper-style dashboard with Today / Project / Activity / Brain / Brief / **Week** / **Hold** / Studio tabs.

## Commands

- **`/todos-init`** — bootstrap a project: find the PRD/scope doc, extract feature list, audit the codebase for what's already built, mark MVP/95%/100% stages, dependency-order everything, write `.claude/todos.json`.
- **`/todos-plan`** — batch-plan a scope of tasks up front (backend-first): walk the scoped tasks in dependency order, surface every ambiguity once, pin each task's contract, and print a runway report of how far the build can go autonomously before you're needed again. Writes `plan` + `plan_question` rows to `brain.db`; mirrors `plan_status`/`plan_id` onto the board.
- **`/todos-build`** — execute the planned tasks autonomously (backend-first): build each in dependency order off its pinned contract, run the backend suite, and halt + flag downstream the moment tests fail. Workflow-tool engine by default (isolated agent per task), `--mode subagent` for a watchable foreground run; checkpoint-commits per phase on a `todos-build/*` branch. Re-running resumes (done tasks skipped).
- **`/todos-brief`** — roll the built backend's pinned contracts into `.claude/docs/frontend-brief.md`: a deduped, render-ready manifest of entities, endpoints (with response shapes), and events, grouped by feature. The bridge from a finished backend to the frontend pass — the input `/todos-design` consumes. **`--from-code`** reverse-extracts the brief from an existing Laravel+Inertia app (a legacy / UX-rescue with no `/todos-plan` history) by reading routes + `Inertia::render` props + FormRequests + Resources.
- **`/todos-design`** — run the frontend design pass: turn `frontend-brief.md` into a built, dual-audited UI using the four design pillars in `lib/frontend/` (process / knowledge / distinctive / effortless-ux). Foreground + interactive — proposes a direction and confirms it with you, derives nav from the brief's feature groups, builds each feature's screens **composed from existing components**, then halts at two independent gates: craft ("would I sign my name to this?") and usability ("can a stranger operate it?" — zero sev-3/4 to ship). Renders + eyeballs the pixels, saves `.interface-design/system.md`. Systems/dashboards first; marketing briefs hand off to `taste-skill` / `/frontend-design`. **`--rescue`** redesigns an *existing* app on its untouched backend (the "client says it's too complicated" case): captures a pain inventory, re-flows the presentation, and **preserves the route/prop/field seam** (rename a prop key → blank page).
- **`/todos-sync`** — mid-work check-in: diff the project against the saved board, propose updates in batches, then run brain-capture stages A–D to surface new decisions / business rules / contradictions / scope changes from recent conversation + git.
- **`/todos-close`** — register completion: sync first, then either close the active phase or the whole project (auto-detected). Writes a phase summary or appends to `CHANGELOG.md`.
- **`/todos-week`** — plan the implementation week: interview the user, pull open tasks across every tracked project, propose a day-by-day Mon–Sun schedule, write the global `schedule.json` the Week tab reads. Cross-project — not run inside one project.
- **`/todos-dashboard`** — start the local Bun server (if not already running) and open `http://localhost:5757`.
- **`/brain-add`** — single-shot capture into `.claude/brain.db`: log a decision, business rule, contradiction, scope change, feedback item, stakeholder, or architecture note without running a full sync.

## Planning a backend autonomously (`/todos-plan`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gonoff/claude-todos-skill](https://github.com/gonoff/claude-todos-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
