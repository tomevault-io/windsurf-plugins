---
trigger: always_on
description: Originate gated Delivery and Interview candidates, or coordinate selected Program Delivery as FRAME -> CHOOSE -> START -> ADVISE -> VERIFY -> ADVANCE. Human-named work bypasses candidate selection. Agent-originated work keeps read-only ORIENT, 1-3 candidates, mandatory human selection, reject-all, and no auto-pick path. Each Program slice uses at most one fresh direct-user Delivery task. Automatic launch requires surface lifecycle support to create, identify, inspect, and message that task; othe
---


# Nauro loop skill

Run gated work origination and coordinate selected Program Delivery. The Program state machine is `FRAME -> CHOOSE -> START -> ADVISE -> VERIFY -> ADVANCE`. The coordinator owns the verified frame, sequence, handoff, advisory review, integration verification, and next recommendation. It does not implement, approve, file project truth, push, create a pull request, merge, or start the next slice.

User-named work enters as `HUMAN-SELECTED`. Agent-originated work keeps read-only ORIENT, 1-3 ranked Delivery and Interview candidates, mandatory SELECT, reject all, and no auto-pick path. Interview remains explicit, separate, and non-authoritative. Synchronous non-program Delivery stays outside the Program state machine and keeps the current `nauro-ship-task` chain.

## Authority boundary

These are authority rules. MCP or shell write paths may exist, but they do not grant the coordinator authority to use them for Program Delivery.

- The loop cannot automatically change project truth or file decisions. Ordinary prompts, Interview results, Delivery handoffs, and program handbacks create no automatic store artifacts. The scheduled ORIENT selection-checkpoint writes are the narrow exception: they create only the existing SELECT checkpoint and pointer through the filesystem and `nauro sync` mechanism defined below. A selected Interview returns candidate shared understanding only. After the interview, an explicitly approved `propose_decision`, `update_state`, or `flag_question` payload may be executed through `/nauro-interview` according to its contract and separate later approval gates. Those writes belong to the interview contract, not loop authority.
- The coordinator may inspect pull-request and merge state through read-only channels, including `gh pr view` or an equivalent authenticated read channel, only when a required PR-backed repository-anchor check needs it. This includes ORIENT, Resume, replacement-coordinator recovery, and VERIFY. Read-only inspection does not grant publication authority. The coordinator never pushes, creates or edits a pull request, merges, or performs another publication mutation. Only the direct-user Delivery task may perform approved publication work through its own gates.
- The loop is NOT a "keep moving" override of any inner gate. A standing "keep going" or auto-mode directive does not clear the SELECT gate, the plan gate, a tech-lead pause, or the push gate. The loop exists to repeat the gated chain, not to bypass it.
- SELECT is never auto-picked. Neither entry mode picks for the human, not even when exactly one candidate ranks or when a scheduled continuation has one surviving candidate. The synchronous mode surfaces SELECT in the parent session; the scheduled mode parks a SELECT checkpoint and exits before any gate; the resume continuation surfaces SELECT to the human. No path resolves SELECT without the human.
- A hidden child, ordinary subagent, generic agent, or persistent Delivery child is not a substitute for a fresh direct-user Delivery task.

## FRAME

Verify the program goal, ordered review-sized slices, dependencies, expected behavioral state, repository, and current `origin/main`. For `HUMAN-SELECTED` input, FRAME also verifies the named invariant. Agent-originated FRAME does not require a selected invariant before CHOOSE. Record the repository anchors and whether an active Delivery identity already exists. Keep this evidence internal unless it changes the user's choice, authority, risk, or next required action. A conflict or missing anchor holds the program.

## CHOOSE

If the user named the work, mark it `HUMAN-SELECTED` and preserve the user's exact named work. `HUMAN-SELECTED` skips candidate origination, ORIENT, and SELECT. Do not add a redundant choice gate.

Agent-originated work enters ORIENT. Its selection flow is `ORIENT -> SELECT -> ROUTE`. It must complete mandatory SELECT before routing. The scheduled path preserves its durable SELECT checkpoint. Program mode remains an explicit opt-in Delivery policy. Interview can be chosen, but it stays in the live coordinator and does not enter Program Delivery.

### ORIENT: mine the store, read-only

ORIENT writes nothing to doctrine. It reuses the Resume mining logic to read the project's current state and assemble candidate work:

- `get_context(level="L0")` for the concise project summary — current state, the top open questions, and last-10 active-decision summaries. That is enough to rank candidates against current direction; ORIENT does not need full decision bodies to compose the set, so it takes the cheaper L0 projection rather than the larger working set.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nauro-AI/nauro](https://github.com/Nauro-AI/nauro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
