---
trigger: always_on
description: Provides the caller template (`templates/pr-relevance-caller.yml`) that any repository
---

# Agent Skills

## Audience

The skills and agents in this repository are consumed operationally by agentic frameworks (AI coding agents, copilots, and autonomous developer tools).
Every piece of guidance must be written so that an agent can act on it without human interpretation.

When writing or editing content, follow these principles:

- **Be prescriptive, not descriptive.**
  Tell the agent what to do, not explain concepts.
- **Make decisions enumerable.**
  Provide numbered decision processes, lookup tables, or explicit criteria.
- **Include code examples for every actionable rule.**
  Show both correct and incorrect patterns.
- **Avoid subjective conditions.**
  State concrete, testable criteria.
- **Keep rules self-contained.**
  Each file must make sense on its own.

## Repository Structure

Skills live in `skills/<category>/<name>/SKILL.md` across 7 categories.
Agents live in `agents/` since they need their own model and tool configuration.

Type markers (by primary entry point — all three are technically model-invocable via the `Skill()` tool when `disable-model-invocation: false`): `auto` = description aggressively auto-triggers on natural language; `/` = primary entry is the slash command, description does not auto-trigger; `Skill()` = primary entry is being called by another skill / workflow.

### `workflow/` — end-to-end orchestrators

- `autonomous-workflow` (`auto`) — phase-based feature delivery 0–7. Opt-in `aw` dispatcher detects tier (Micro/Lite/Full) and routes single-pass vs the planner→executor split (Full only). Two-tier self-improvement hoisted to the dispatcher (universal): fast episodic-lessons tier (LoreKit `loop::aw-lessons`) promotes to the gated `diagnose` slow tier at `seen_count ≥ 3`. Loop: [`rules/self-improvement-loop.md`](./skills/workflow/autonomous-workflow/rules/self-improvement-loop.md). Plan-quality gates (v3.15): Phase 0 restate-and-diff + missing-information gate (`blocking` halts even under `--no-confirm`), Phase 1 Existing Code Survey per planned `create` (anti-reinvention, `confidence` rule #10) + `AC-{n}`/`(covers: R{m})` requirement traceability (rule #9), and an executable `checks.yaml` acceptance artifact (rule #11) the executor's Phase 4 loop gates on mechanically — definitions executor-immutable, check-gaming forbidden, `unsatisfiable` abort affordance. Artifact lightening (v3.18): `checks.yaml` is the primary living contract and `plan.md` a lean handoff document the executor writes drift back into (Phase 3); `plan.v{N}.md` snapshots are opt-in (`aw-create-plan`'s `snapshot` arg), not default; the "No AI co-author tags" rule was removed. Research basis: [`references/planning-quality-research.md`](./skills/workflow/autonomous-workflow/references/planning-quality-research.md). Design intent: [`workflow/autonomous-workflow/CLAUDE.md`](./skills/workflow/autonomous-workflow/CLAUDE.md)
- `aw-create-plan` (`Skill()`) — writes `plan.md` + `checks.yaml`; immutable `plan.v{N}.md` snapshots are opt-in (`snapshot` arg). `aw-create-walkthrough`, `aw-review-quality-gate` (`Skill()`) — autonomous-workflow companions
- `batch-linear-tickets` (`/`) — batch-analyze Linear tickets by dispatching `linear-ticket-investigator` (plus `holistic-analysis` for bug tickets) per ticket, then fan out fixes; requires Linear MCP. Self-improvement: `batch-lessons` fast tier (read Phase 1 / write Phase 5) for classification + correlation; inherits `aw-lessons` via the planner/executor fan-out; promotes to `diagnose`
- `fix-bug` (`/`) — single-bug pipeline phases 0–8. Flags: `--analyse-only`, `--force-holistic`. Self-improvement: `fix-bug-lessons` fast tier (read Phase 0.5 / write Phase 5·7·8) for its diagnostic phases; inherits `aw-lessons` via `aw-executor`; promotes to `diagnose`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mthines/agent-skills](https://github.com/mthines/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
