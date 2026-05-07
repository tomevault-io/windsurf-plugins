---
trigger: always_on
description: This repository is the standalone home for the GodMode OpenClaw plugin.
---

# GodMode Plugin - Agent Context

This repository is the standalone home for the GodMode OpenClaw plugin.

## Product Architecture — READ FIRST

**Read `docs/GODMODE-META-ARCHITECTURE.md` before making any architectural decisions.** It is the definitive blueprint for how GodMode gets built.

**Read `HARNESS.md` for agent workflow rules** — branching, building, merging, handing off. Follow it.

**Read `TEAM-WORKFLOW.md` for team collaboration rules** — slash commands (`/bug`, `/fix`, `/pr-review`, `/sync`), branch protection, and CI enforcement.

### What GodMode Is
GodMode is a deeply contextual personal AI ally that manages a swarm of agents. The ally is 80% of the value (deep context, coworking in chat). Agent delegation is 20%. GodMode is the conductor, not the orchestra — it connects to the user's existing tools, never rebuilds them.

### The Three Golden Rules
1. **Code as little as possible.** Can this be a file (persona, skill, recipe)? If yes, don't write TypeScript. Only write engine code for: ally identity, context stack, orchestration, queue processing, trust tracking.
2. **Conduct, don't rebuild.** NEVER build a CRM, file explorer, project management tool, email client, calendar app, note editor, code editor, social media manager, analytics platform, or chat platform. The ally connects to the user's existing tools via API/MCP.
3. **Meta-agent pattern.** The ally crafts precise prompts for sub-agents. Quality scales through prompt quality, not more code.

### Scope Boundaries — NEVER Build These
- CRM / contacts manager → plug into Apple Contacts, HubSpot, Google
- File explorer / storage → ally reads/writes files via tools
- Project management (boards, sprints, dependencies) → ally reads/writes ClickUp, Linear, Asana via API
- Email client → ally reads email via integration
- Calendar app → Today tab shows schedule, don't rebuild calendar
- Note-taking app → Obsidian IS the note-taking app
- Code editor → VS Code, Cursor exist
- Social media manager → content-writer persona creates content, user posts via their tool

### Task System Scope
GodMode tasks = flat operational notepad (title, due date, status, workspace). No hierarchy, no subtasks, no boards. Big projects = markdown artifacts the ally creates. Ally bridges to external PM tools, never mirrors them.

### The 6-Tab UI Baseline
Chat → Today → Work → Second Brain → Dashboards → Settings. Everything else hides behind Settings. Work tab shows GodMode artifacts ONLY: sessions, agent outputs, tasks, artifacts, skills, workspace memory.

### Anti-Bloat Rule
Nothing gets permanent context injection. New capabilities are files (personas, skills) or conditional context (state-checked, injected only when relevant). The only always-on injection: ally identity (~30 lines) + awareness snapshot (~50 lines).

## Default Development Workflow — MANDATORY

When the user gives a task, plan, or big prompt, follow this pipeline automatically. Do NOT skip steps. Invoke each skill via the Skill tool at the appropriate phase.

### Phase 0: Understand
- **Any creative/design work** → invoke `/brainstorming` FIRST. Explore intent before touching code.
- **Big task or spec** → invoke `/writing-plans` to produce a plan in `docs/plans/`.
- **Vague idea** → invoke `/gstack-office-hours` (builder mode) to shape it into a design doc.

### Phase 1: Review the Plan
- **Every plan gets reviewed.** Invoke `/gstack-plan-eng-review` for architecture, data flow, edge cases.
- **If plan touches UI** → also invoke `/gstack-plan-design-review` for interaction states, responsive, accessibility.
- **If plan is strategic/scope-heavy** → also invoke `/gstack-plan-ceo-review` for scope challenge.
- Fix the plan based on review feedback before writing any code.

### Phase 2: Safety Rails
- Invoke `/gstack-guard` at session start for destructive command warnings + edit boundary enforcement.
- If working in a specific directory, invoke `/gstack-freeze` to lock edits to that scope.

### Phase 3: Implement
- **Multiple independent tasks** → invoke `/dispatching-parallel-agents` to parallelize.
- **Sequential tasks from a plan** → invoke `/executing-plans` or `/subagent-driven-development`.
- **Writing new features** → follow `/test-driven-development` (failing test first, always).
- **Hit a bug** → invoke `/systematic-debugging`. NEVER guess at fixes.

### Phase 4: QA
- After implementation, invoke `/gstack-qa` to systematically test and fix issues.
- For report-only (no fixes), use `/gstack-qa-only`.
- If the project has a live UI, invoke `/gstack-browse` for headless browser QA.

### Phase 5: Review & Ship
- Invoke `/gstack-review` for pre-landing code review (SQL safety, trust boundaries, side effects).
- Invoke `/verification-before-completion` before claiming anything is done.
- Invoke `/gstack-ship` for the full ship workflow (merge base → test → version bump → changelog → PR).

### Phase 6: Post-Ship
- Invoke `/gstack-document-release` to sync all docs to match what shipped.
- Invoke `/gstack-retro` periodically (weekly or after big features) for retrospective.

### Quick Reference — When to Use What
| Situation | Skill |
|---|---|
| "Build X" / "Add Y" / new feature | brainstorming → writing-plans → plan-eng-review → implement → qa → ship |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GodMode-Team/godmode](https://github.com/GodMode-Team/godmode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
