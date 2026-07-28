---
trigger: always_on
description: <!-- BEGIN MANAGED: parallelism-and-file-discipline -->
---


<!-- BEGIN MANAGED: parallelism-and-file-discipline -->
## Parallelism and file discipline

- `isolation:none + enforcement:strict + file-disjoint W2` is the proven default pattern across 15+ consecutive green sessions. Do not deviate without an explicit reason.
- File-disjoint `allowedPaths` per agent is enforced at the prompt level regardless of worktree isolation mode. When worktree isolation is dropped due to RAM pressure, allowedPaths must still be strictly disjoint.
- When 2+ planned tasks share >50% file scope, merge them into one agent before W2 dispatch to avoid parallel-write conflicts.
- When 2 agents both want to edit a shared file, have one localize its changes to a sister file — prefer clean separation over coord-merge work.
- Do not dispatch concurrent agents to edit CLAUDE.md; collect proposed YAML additions verbatim in agent reports and apply them coord-direct in W5 Finalization.
<!-- END MANAGED: parallelism-and-file-discipline -->

<!-- BEGIN MANAGED: wave-execution -->
## Wave execution

- 5W structure default: Discovery → Impl-Core → Impl-Polish → Quality → Finalization.
- Housekeeping sessions: single-wave Express Path, 0-6 agents, coordinator-direct. No multi-wave planning needed.
- 5W×6A thin-slice epics with shipped substrate: W1 6 parallel Explore, W2 6 file-disjoint code-implementers, W3 typically reduces to 4 after W2 absorption, W4 test-writers + security-reviewer, W5 2-3 agents.
- Inter-wave Quality-Lite gate after Impl-Core must include `npm test` when production fixes touch files with adjacent test files — typecheck+lint alone is insufficient.
- When session-reviewer reports BLOCK at end of W2, add the fix as a new agent in W3 (Impl-Polish); do not restart W2.
- Test-writers must verify both `npm test` (all tests pass) AND `npm run lint` (zero lint errors) before reporting done. Lint-only verification allows stylistic regressions to slip to Full Gate.
- When a test-writer agent runs tests against production code, then mutates the SUT to a known-broken state and re-runs to observe failure, this falsifiability cycle proves the test catches the regression it claims to cover. Mutation+revert cycles are expected in test delivery.
- When a wave ships a fix for a recurring anti-pattern, run a pattern-replication audit on the rest of the diff before W4 closes — the agent who just fixed the bug is the most likely to re-introduce it elsewhere in the same change set. A single-agent review misses the recurrence; a multi-reviewer W4 panel catches it. Add a 1-line grep of the diff for other instances of the just-fixed pattern.
- MEDIUM findings discovered in-session (during W3/W4) should be folded and documented in STATE.md Deviations rather than filed as carryover issues. Exceptions: MEDIUM findings that require redesign (scope change beyond the current agent's file boundary) stay as blockers.
<!-- END MANAGED: wave-execution -->

<!-- BEGIN MANAGED: discovery-and-scope-adjustment -->
## Discovery and scope adjustment

- W1 Discovery findings that warrant scope reduction or expansion must surface via AUQ before W2 dispatch.
- When Discovery reveals the planned work was already shipped by a prior session, immediately reduce scope rather than re-implementing.
- For sessions where issue bodies claim external submission status (e.g., "awesome-list"), W1 must web-fetch the upstream list to confirm current state before dispatching W2 work.
- W1 agents must grep-verify all file-location claims and API-shape assumptions from the issue body before W2 scope takes shape. Pattern: issue claims "function X exported from module Y" → grep Y for the export; issue lists N callsites → grep the repo to verify only those N exist. Pre-dispatch verification catches mismatches (CLI-only vs importable, file renames, missing exports, SUT mis-attribution) before W2 wastes effort. Quote the exact grep pattern, file scope, and result count in the Discovery report.
- When Discovery grep-verifies that an issue AC is factually impossible or wrong (e.g., AC says "filter in file X" but grep proves file X has 0 references to the filter), the coordinator MUST surface the ambiguity via AUQ BEFORE Impl-Core dispatches against the wrong locus. The agent role is to report the contradiction with evidence; the coordinator decides how to proceed (adapt AC, reduce scope, ask user for clarification). Never let Impl agents silently resolve factual contradictions.
<!-- END MANAGED: discovery-and-scope-adjustment -->

<!-- BEGIN MANAGED: architecture-and-code-patterns -->
## Architecture and code patterns

- When splitting a parent module into child submodules, extract schema/leaf types to a sibling module first. The dependency graph must be unidirectional: schema → io/filters → barrel. A barrel that re-exports children that import from the parent creates a real ESM circular import.
- The file-conflict matrix (D5 Discovery) checks file overlap, not dependency direction. Architect-reviewer is required to catch circular-import risks from module splits.
- Production modules that may be `vi.mock`ed in sibling test files must use lazy dynamic imports (`await import(...)`) instead of top-level static imports. Top-level static imports cache the real module in the vitest fork pool, preventing mock interception.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kanevry/session-orchestrator](https://github.com/Kanevry/session-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
