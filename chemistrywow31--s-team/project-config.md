---
trigger: always_on
description: Produce professional presentations backed by verified research, expert domain knowledge, and implementable technical solutions. Supported deliverables include HTML slides, PowerPoint (`.pptx`), PDF, and Web POC prototypes. Every presentation must pass quality review before delivery.
---

# Presentation Studio Codex Runtime

## Team Objective

Produce professional presentations backed by verified research, expert domain knowledge, and implementable technical solutions. Supported deliverables include HTML slides, PowerPoint (`.pptx`), PDF, and Web POC prototypes. Every presentation must pass quality review before delivery.

## Platform Doctrine

- **Canonical source design: `.claude/` and `CLAUDE.md`.** All design changes are authored there first.
- **This Codex tree is generated parity**: `AGENTS.md`, `agents/**/*.toml`, `.codex/rules/`, `.codex/skills/`, and `.agents/skills/` are regenerated from the Claude tree per `.codex/docs/format-mapping.md`. Do not hand-edit the generated surfaces to change design; change `.claude/` and regenerate.
- Codex-only additions that have no Claude source (the 7 Codex-native rules in `.codex/rules/`, including `presentation-visual-asset-standard.md`, and the Codex runtime registry `.codex/config.toml`) are preserved across regenerations.
- Mapping artifacts: `.codex/docs/format-mapping.md` and `.codex/docs/format-mapping.manifest.yaml`.

## Deployment Mode: Coordinator Dispatch (Subagent-Style)

One Coordinator orchestrates all specialists through isolated `spawn_agent` dispatches within a single Codex session. There is no shared task ledger and no peer messaging.

- **The Coordinator runs in the MAIN session.** Invoking `$boss` makes the current session adopt `agents/coordinator.toml`'s `developer_instructions` as its playbook. Never spawn the Coordinator as a subagent — a spawned coordinator cannot dispatch further agents and cannot converse with the user. Production evidence: `.worklog/202605/qic-ai-travel-review-tech-deck/phase-6-5-process-review/process-review.md` records a run degraded to single-agent mode for exactly this reason.
- **Specialists are dispatched via `spawn_agent`** from the project-local registry in `.codex/config.toml`, one dispatch per unit of work, each starting with fresh context. A specialist that needs user input ends its run with `STATUS: NEEDS_CONTEXT` and a `QUESTIONS:` block; the Coordinator relays the questions to the user, logs the exchange to the task worklog, and re-dispatches with the answers.
- **Parallel groups** run as concurrent spawns in the same Coordinator turn: Phase 2 (Investigative Researcher ∥ Domain Expert) and Phase 5 (Visual Designer ∥ Web Developer).
- **Handoffs flow through the Coordinator.** Every dispatch carries the worklog path, upstream reference paths, task scope, acceptance criteria, and a scope fence. Every return follows the EC-1 report schema in `.codex/rules/execution-contract.md`. See `.codex/rules/communication-protocol.md`.

Prerequisites: open Codex from this `presentation-studio` project root; use the project-local `.codex/config.toml` (no `~/.codex/config.toml` changes required); enter the workflow through `$boss`.

## Agent Roster

| Agent | Config | Phase | Role |
| --- | --- | --- | --- |
| Coordinator | `agents/coordinator.toml` | all | Main-session playbook; intake, dispatch, gates, routing (never spawned) |
| Investigative Researcher | `agents/research/investigative-researcher.toml` | 2 | Source discovery, credibility rating, source registry |
| Domain Expert | `agents/research/domain-expert.toml` | 2 | Domain analysis, frameworks, knowledge gaps |
| Presentation Architect | `agents/planning/presentation-architect.toml` | 3 | Narrative structure and slide-by-slide outline |
| Technical Architect | `agents/planning/technical-architect.toml` | 3 | Technical solutions, feasibility, POC specs |
| Presentation Writer | `agents/production/presentation-writer.toml` | 4 | Slide copy and speaker notes |
| Visual Designer | `agents/production/visual-designer.toml` | 5 | Style guide, effect specs, imagery, layouts |
| Web Developer | `agents/production/web-developer.toml` | 5 | HTML/PPTX/PDF/POC builds + layout and render gates |
| QA Reviewer | `agents/quality/qa-reviewer.toml` | 6 | Deliverable quality review and issue routing |
| Process Reviewer | `agents/review/process-reviewer.toml` | 6.5 | Coordination retrospective (distinct from QA) |

## Workflow Phases

```text
Phase 1: Requirements Intake        -> Coordinator
Phase 2: Research (parallel)         -> Investigative Researcher || Domain Expert
Phase 3: Architecture Planning       -> Presentation Architect + Technical Architect
Phase 4: Content Writing             -> Presentation Writer
Phase 5: Visual & Build (parallel)   -> Visual Designer || Web Developer
Phase 6: Quality Review              -> QA Reviewer
Phase 6.5: Process Review            -> Process Reviewer
Phase 7: Revision & Delivery         -> Route back to relevant agent(s) based on review feedback
```

## Universal Rules

### Communication Language

Communicate in the user's language. Detect and match the language the user uses. Technical terms may remain in English.

### Output Directory Convention


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chemistrywow31/S-Team](https://github.com/chemistrywow31/S-Team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
