---
trigger: always_on
description: This file is host-neutral and applies to **every phase**. Both entrypoints (`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex) point here, and phase prompts reference it as `prompts/conventions.md`. When a phase prompt and this file disagree on state schema, enums, or round numbering, **this file wins**.
---

# Shared Conventions (All Phases, All Hosts)

This file is host-neutral and applies to **every phase**. Both entrypoints (`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex) point here, and phase prompts reference it as `prompts/conventions.md`. When a phase prompt and this file disagree on state schema, enums, or round numbering, **this file wins**.

Request routing and execution behavior are owned by `prompts/agent_workflow.md`. Its autonomy rules do not override this file's state, lock, terminal-status, or recovery gates.

## Startup Protocol

Run this protocol for research execution, after request routing in `prompts/agent_workflow.md`.

1. **Check setup and security schema before project discovery:** inspect only the structure of `.rev2agent_config.json`. Read `prompts/00_setup.md` and complete setup/migration if config is absent, `version` is not `2`, a legacy `api_key` value field exists, or required version-2 privacy fields are invalid. Never use a legacy credential value.
2. **Select a project:** research projects live in subdirectories of the repository root. Use an already selected project; validate its path and state rather than asking again. Otherwise scan for project directories containing `.research_state.json`, excluding `_new_project_draft/`. If that draft exists, offer resume/discard under Phase 1. List available projects and ask which to resume or whether to start new. An empty list or missing selected project does not authorize a replacement: start Phase 1 only when new research is requested.
3. **Resume from state:** `{project_dir}/.research_state.json` is authoritative, and its `project_dir` is the base for every project operation. Check `project_status` before dispatching on `phase_status` and before creating a session lock:
   - `completed`: report final artifacts and do not advance.
   - `archived`: report that it is archived and do not mutate or advance.
   Finish terminal-status responses without soliciting reactivation. For active projects, apply the State Write Rules and Error Recovery lock/kill-flag checks below. Then read the current phase prompt and dispatch:
   - `not_started`: begin the phase.
   - `in_progress`: check persisted artifacts and any running process, then resume the remaining work.
   - `waiting_for_user`: match the latest reply to the pending artifact/decision; record exact existing approval and complete the transition checklist, or ask for the missing decision.
   - `completed`: advance using the owning phase's state update and transition checklist.
   - `failed`: diagnose and recover under Error Recovery.

## Phase Overview

Read the owning prompt before phase work. It defines the required outputs, scientific decisions, and next transition; this index does not authorize skipping them.

| Phase | Prompt |
|-------|--------|
| 0 — Setup | `prompts/00_setup.md` |
| 1 — Topic Interview | `prompts/01_interview.md` |
| 2 — Literature Search | `prompts/02_literature_search.md` |
| 3 — Research Plan | `prompts/03_research_plan.md` |
| 4 — Experiment Design | `prompts/04_experiment_design.md` |
| 5 — Experiment Execution | `prompts/05_experiment_execution.md` |
| 6 — Result Analysis and Round Planning | `prompts/06_result_analysis.md` |
| 7 — Manuscript Writing | `prompts/07_manuscript_writing.md` |
| 8 — Manuscript Review | `prompts/08_manuscript_review.md` |

## State File: `{project_dir}/.research_state.json`

The single source of truth for session resumption. Schema:

```json
{
  "project_dir": "semantic_segmentation",
  "current_phase": 1,
  "sub_step": null,
  "current_round": 0,
  "current_round_short_name": "",
  "phase_status": "in_progress",
  "project_status": "active",
  "created_at": "2026-02-12T10:00:00Z",
  "updated_at": "2026-02-12T10:00:00Z",
  "topic": {
    "broad_topic": "",
    "specific_topic": "",
    "research_question": "",
    "positioning": "",
    "target_venue": "",
    "target_dataset": [],
    "metrics": []
  },
  "literature": {"papers_reviewed": [], "future_work_ideas": [], "selected_direction": ""},
  "experiment": {
    "plan": "",
    "scripts_ready": false,
    "status": "not_started",
    "checkpoints": [],
    "estimated_time_hours": 0,
    "hardware_requirements": {},
    "active_runs": []
  },
  "results": {"raw_results_path": "", "analysis_summary": "", "user_confirmed": false},
  "manuscript": {"title": "", "abstract": "", "latex_path": "", "figures": [], "status": "not_started"},
  "phase_history": []
}
```

## Enums (use these values EXACTLY — no improvisation)

| Field | Allowed values |
|-------|----------------|
| `phase_status` | `not_started` \| `in_progress` \| `waiting_for_user` \| `completed` \| `failed` |
| `project_status` | `active` \| `completed` \| `archived` |
| `experiment.status` | `not_started` \| `running` \| `completed` \| `failed` |
| `experiment.active_runs[].status` | `running` \| `completed` \| `failed` |
| `manuscript.status` | `not_started` \| `in_progress` \| `draft_complete` \| `in_review` \| `final` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dalbom/rev2agent](https://github.com/dalbom/rev2agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
