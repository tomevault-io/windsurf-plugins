---
trigger: always_on
description: This file defines implementation rules for this repository. Higher sections are intentionally higher priority.
---

# AGENTS.md

This file defines implementation rules for this repository. Higher sections are intentionally higher priority.

## Required Startup Reading (Mandatory)

- Read `GODOT_BEST_PRACTICES.md` at session startup before planning or implementation.
- Treat `GODOT_BEST_PRACTICES.md` as required process, not optional guidance.
- Godot operational/testing/validation and invocation rules are canonical in `GODOT_BEST_PRACTICES.md` and are fully enforceable.
- Validation order default is mandatory: run a real non-headless launch first to surface parser/runtime scene errors early, then run headless harness sweeps.

## Sub-Agent-First Execution Model (Highest Priority)

- Planning-first, impact-first execution: each wave starts with a concrete target and explicit priority order.
- Initial investigation is mandatory: launch a planning sub-agent to scan current state and risks before any implementation edits.
- The main thread is executive only: coordination of user communication, architecture decisions, and lane orchestration.
- **Main-thread hard rule**: The main thread MUST NOT perform implementation edits. That means no `apply_patch`, no file edits, no config rewrites, and no direct code edits in source/config docs (except this instruction file when updating process).
- **Main-thread hard rule**: The main thread MUST NOT perform planning, merge/review coordination, validation planning, or verification tasks if a sub-agent can do them.
- **User override rule**: If the user explicitly requests main-thread execution (for example, “do this yourself without spawning a sub-agent”), that request overrides main-thread behavior rules in this file for the scoped task, as long as higher-priority system/developer safety constraints are still respected.
- **Allowed on main thread only**: user-facing communication, high-level escalation, and explicit lane orchestration only.
- **Allowed to read on main thread**: repository state checks and agent outputs for context.
- **Implementation rule**: all source/config/behavior edits are executed by spawned `worker` lanes.
- **Verification rule**: all verification for contract/native changes is executed by spawned `validation`/`Test-Infrastructure` lanes.
- Use sub-agents for both planning and execution (mandatory):
  - validation and verification should also be assigned explicitly to validation lanes for contract-heavy or native-path changes.
  - planning must always be done by a planning sub-agent that maps scope, owners, acceptance criteria, and risk calls before implementation starts.
  - spawn implementation/validation sub-agents whenever responsibilities can be split safely.
- Workflow loop:
  0. Before any implementation edit, estimate final file size impact and enforce file-size preconditions.
  - If any in-scope edit targets a source/config file at or above 900 lines, execution must be split before code changes begin.
  - If predicted delta pushes a file over 1000 lines, create or reuse a dedicated helper/source split lane immediately and move non-call-site logic there.
  1. Update `ARCHITECTURE_PLAN.md` first with priority band (P0/P1/P2), owners, and acceptance criteria.
  2. Launch one or more planning sub-agents for decomposition + coupling-risk assessment.
  3. Launch scoped implementation and validation sub-agents in parallel.
  4. Keep main-thread focus on coordination and escalation while agents own execution, integration/deconflict, verification, and commit/push flow.
  5. Proactively close stale or completed sub-agents immediately when they finish; do not defer. This is mandatory to preserve agent slots and prevent forced context resets in future turns.
  6. Track active agent slots; if only 4 or fewer slots remain, notify the user immediately and proceed conservatively with spawn decisions.
  7. Deconflict overlaps immediately and merge outputs as soon as they land.
  8. Run targeted verification on the highest-impact wave via validation lanes.
  9. Keep plan/status updates synchronized as scoped changes are committed via sub-agent lanes.
- Ask user questions only when an architectural or requirement decision is truly ambiguous, through the coordinating sub-agent.
- If a main-thread implementation edit is accidentally initiated, stop, report the violation immediately, and do not continue until explicit user confirmation to resume via lanes.
- If a main-thread planning, merge/review coordination, or validation activity is accidentally initiated when a sub-agent can execute it, stop and reassign to sub-agent lanes immediately.
- **Agent lifecycle rule (critical)**: The main thread must close completed/stale agents (`close_agent`) as soon as possible after they are no longer active. Delayed closure is a coordination failure with direct operational impact (slot pressure and workflow fragmentation).

### Additional Sub-Agent Lanes (including but not limited to)

- GitHub-Operations lane: manage branch hygiene, PR lifecycle (`gh` status/labels/reviewers/checks), commit/push flow, and PR synchronization; owns status/error triage and merge blockers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adammikulis/local-agents](https://github.com/adammikulis/local-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
