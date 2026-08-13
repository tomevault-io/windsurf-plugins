---
trigger: always_on
description: The following directories contain core TCE infrastructure. NEVER edit, revert, or modify files in these paths, even if you believe there is a bug:
---

# Open Timeline Engine — Executor Policy

## CRITICAL: Protected Files — DO NOT MODIFY
The following directories contain core TCE infrastructure. NEVER edit, revert, or modify files in these paths, even if you believe there is a bug:
- `shared/tce_shared/`
- `services/tce_api/`
- `services/tce_lite_api/`
- `services/tce_mcp/`
- `scripts/`
- `infra/`

If you find a bug in these paths, report it to the user instead of changing it, unless the user explicitly asks to override this protection.

## Required tool-calling behavior

### Session management
1. Use one stable `session_id` per conversation and reuse it on every call.
2. Always pass `activation_mode_default="takeover"` on every `tce.takeover_step` call.
2a. On the first takeover call in a repository, pass `app_context={"project_root":"<absolute repo root>","project":"<repo name>"}`. Re-send it when the active repository changes; TCE binds it to the session for later calls.

### State resolution (history-driven)
3. Determine takeover state from conversation history ordering, not local flags:
   - If any activation phrase appears and no stand-down appears after it => active.
   - If a stand-down phrase appears after the last activation => inactive.
4. Do not rely on persistent local flags like `tce_takeover_active`.

### When to call takeover tools
5. On activation turn, call `tce.takeover_step`.
6. While active, call `tce.takeover_step` on every user message.
7. On stand-down turn, call `tce.reset_takeover_state(session_id)` and stop takeover calls afterward.
8. While inactive, do not call takeover tools in normal chat flow.

### Activation and stop phrases
- Activation: `beru take over`, `beru takeover`, `beru suggest`, `igris take over`, `kurama take over`, plus custom phrases.
- Stop: `beru stand down`, `shadow stand down`, `advisor stand down`, `kurama stand down`, plus custom stop phrases.

### Handling the result
9. If `persona_ack` exists, output exactly that text first; if `has_directive=true`, immediately continue directive execution in the same turn.
10. If `safety_decision=confirm_required`, show `final_response` and wait for confirmation.
11. If `has_directive=true`, immediately work on `state.takeover_context.objective` using tools. Do not narrate tool payloads.

### V6 directive lifecycle (mandatory)
12. If `execution_permit_required=true` and permit is missing, call `tce.request_execution_permit` immediately.
13. If execution claim is required (or `next_step` says to claim), call `tce.claim_execution` before mutating actions.
14. After execution:
    - Success: call `tce.report_execution(..., state="succeeded")`
    - Failure: call `tce.report_execution(..., state="failed", failure_reason="<reason>")`
    - For completed changes without a takeover directive, call `tce.complete_task` with title, files, decision, outcome/next step, git refs, and anchors before ending the turn.
15. If `retry_scheduled=true`, continue the same objective automatically unless safety requires pause.
16. If `autonomy_notice` is present, call `tce.ack_takeover_notice` unless already acknowledged.

### next_step precedence (critical)
17. If `next_step` contains actionable instructions (for example `AUTONOMOUS MODE PAUSED`, `Call tce.request_execution_permit`, `Call tce.claim_execution`), follow `next_step` first.
18. Natural response is allowed only when:
    - `has_directive=false`
    - `final_response` is null
    - `next_step` is empty or non-actionable
    - no execution lock is present (`pending_execution` absent and `directive_state` not `pending`/`in_progress`)
18a. If takeover is active and execution is locked (`pending_execution` exists or `directive_state` is `pending`/`in_progress`) but `has_directive=false` and `next_step` is empty:
    - call `tce.get_execution_status(session_id)`
    - call `tce.claim_execution(...)` for the latest pending directive
    - continue the same objective; do not emit natural handoff text (for example "If you want, I can do next...")

### Continuation until done or no longer relevant
19. While active, continue autonomously until one is true:
    - goal achieved/completed
    - explicit stand-down phrase
    - existing goal remains low relevance: `selected_goal.selection_score < 0.45` for 3 consecutive evaluation cycles
20. Empty goal queue handling:
    - if `selected_goal` is null or queue is empty, call `tce.takeover_discover_goals`.
    - if goals are discovered, continue with highest-priority goal.
    - if still empty, keep takeover active, ask a concise "what next?" prompt, and do NOT reset session.
21. Vague-response standby criteria while queue remains empty:
    - treat as vague when no actionable verb+target objective can be extracted (for example: "ok", "hmm", "sure", "continue")
    - treat as concrete when a specific task intent is present (for example: "fix dashboard takeover scroll")
    - if vague for 3 consecutive turns: keep session active, switch to suggest/standby, and do NOT call `tce.reset_takeover_state`
22. If `selection_score` is missing for an existing goal, remain active and re-evaluate next cycle.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JOELJOSEPHCHALAKUDY/open-timeline-engine](https://github.com/JOELJOSEPHCHALAKUDY/open-timeline-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
