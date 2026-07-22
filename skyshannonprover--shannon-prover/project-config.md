---
trigger: always_on
description: Event-driven EasyCrypt proof-agent pipeline using managed proof sessions,
---

# Project: Shannon Prover

Event-driven EasyCrypt proof-agent pipeline using managed proof sessions,
structured workspace views, workflow orchestration, replay/audit validation,
and the Knowledge Base.

## Scope Restriction

- Do NOT write files outside this directory.
- Do NOT fetch proof scripts or solutions for eval lemmas from the internet
  (the eval corpus originates from public EasyCrypt developments; fetching
  them defeats eval-mode blinding).

## Managed Prover Architecture

The current proof-agent boundary is manager-owned:

```text
orchestrator
  owns proof-search strategy and tree topology

ProofNodeManager
  the only manager visible to agent/orchestrator

  ReplSessionManager
    internal: EasyCrypt REPL/session lifecycle

  WorkspaceViewManager
    internal: ProofContextView -> ProverWorkspaceView projection

agent
  calls submit_proof_intent and sees one ProverWorkspaceView
  chooses proof-level intents only
```

Ownership rules:

- Orchestrator owns racing/tree policy: spawn, kill, stuck detection,
  capacity, winner selection.
- ProofNodeManager owns the agent turn boundary, metadata binding, view
  refresh, malformed-intent repair, and node health/progress events.
- ReplSessionManager is the only component that starts, checks, commits,
  undoes, restarts, or replays EasyCrypt sessions. Tree sibling creation is
  orchestrator-owned; a child node rebuilds state by replaying a verified
  prefix through its manager-owned session.
- WorkspaceViewManager only projects completed proof snapshots into
  `ProverWorkspaceView`; it does not execute tactics.
- Agent owns proof choices: which tactic to commit, which symbol to look up,
  and which context topic to inspect.

## Agent-Facing Protocol

The prover agent should feel like it is using an IDE. Each turn it receives:

- a short result or repair prompt from the manager
- the latest authoritative `ProverWorkspaceView`

The agent should read:

- `last_result` for the previous manager result, kept brief
- `proof_status` for the minimal lemma/status/view-focus/remaining-goal facts
- `current_goal.lines` for the exact EasyCrypt goal text
- `program_frontier` for visible program/call/frontier structure
- `application_context` for selected handles, requirements, anchors, and
  residual obligations relevant to the current route
- `facts_and_diagnostics` for available evidence, gaps, and recent failures
- `candidate_moves` for neutral proof options
- `call_site_surface`, when present, for L4 call-site facts: live call sites,
  named handles, directly callable handles, frontier-live handles that still
  need binding, frontier blockers, wrapper depth, one-sided call certificate
  evidence, and preview effects
- `seq_cut_surface`, when present, for L4 seq-cut facts: the current seq scope,
  obligation shape, branch focus, and residual frontier after a checked cut
- `pure_tail_surface`, when present, for L4 pure-tail facts after program
  frontier work: sampling side conditions, map-update/projection structure,
  membership decomposition sources, existential witness candidates,
  map-update lookup cases, memory-decoration translation, and visible
  alignment gaps
- `frame_obligation_ledger`, when present, for L4 frame-retention facts:
  frame equalities visible in required context, structural boundary assertions
  that carry them, and reversible boundaries related to possibly dropped facts
  only when those facts are visible in current local goal evidence
- `recovery_diagnosis_surface`, when present, for L4 recovery classification:
  whether the current evidence looks like a boundary repair, call-frontier
  recovery, seq-midpoint repair, local pure-tail surgery, residual program
  surgery, or an ambiguous recovery point
- `structural_checkpoints`, when present, for L4 recovery facts: semantic
  checkpoints such as `before_seq_cut`, `after_seq_opened`,
  `before_branch_work`, `before_call_route`, `after_call_opened`,
  `before_call_obligation_work`, `last_call_site_boundary`, and
  `restore_before_last_rewind`
- `route_replay_memory`, when present, for L4 recovery after checkpoint rewind:
  old discarded route chunks that can be checked in a scratch verifier session
  before any replay commit
- `inspect_lookup_handles` for semantic inspect or lookup requests

`candidate_moves` is information, not an imperative plan. Read
`applicability`, `effect`, `limitations`, and `use_when` before acting. When
`candidate_moves.structural_transitions` is the current-state menu of
reversible proof-phase entrances, such as a checked `wp.` transition that
enters a real post-wp surgery workbench, use one only when its phase is the
phase you want to enter.
If `candidate_moves.route_health` is present, it is the manager's current
route diagnosis for this view only. It may recommend inspect/checkpoint
rewind actions, and may list context lookups under `useful_inspections`. It is
advisory: it can say "a previous boundary may need to be revisited", and may
show a boundary-vs-residual concept diff, but it does not decide how to
strengthen the invariant and is not a proof script or history panel. Route
health entries may include a `recovery_class` and `checkpoint_policy`. Treat
boundary/call-frontier/seq-midpoint classes as evidence that a reversible

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SkyShannonProver/shannon-prover](https://github.com/SkyShannonProver/shannon-prover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
