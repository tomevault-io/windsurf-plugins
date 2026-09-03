---
trigger: always_on
description: kres is a multi-agent kernel code analysis REPL. Three agents collaborate:
---

# kres — Kernel Code Analysis Tool

## Architecture

kres is a multi-agent kernel code analysis REPL. Three agents collaborate:

- **Fast agent** (configurable model): Scopes work, identifies needed source code, builds a structured brief for the slow agent. Runs in task threads.
- **Slow agent** (configurable model): Deep analysis with all context pre-gathered. Thorough findings with file:line citations. Runs in task threads.
- **Main agent** (configurable model): Data retrieval only. Fetches code via semcode MCP, grep, read, git. Runs in service threads spawned per-task.

## Flow

```
User prompt → Task created → Task thread starts
  → Fast agent [round 1]: requests data via followups
  → Service thread: main agent gathers data (semcode/grep/read/git)
  → Fast agent [round 2+]: verifies, requests more or sets ready_for_slow
  → Slow agent: deep analysis with all gathered context
  → Task completes → followups sent through inference for dedup → new todos
```

## Key Design Decisions

### Compatibility Policy
- kres is new enough that there is no backwards-compatibility burden.
- Do not keep duplicate command paths, compatibility aliases, deprecated
  prompt formats, or renamed-file shims unless the current workflow
  actively needs them.
- Workflow-owned commands have exactly one implementation mechanism:
  the JSON workflow under `configs/workflows/` (or the operator's
  `~/.kres/workflows/<name>.json` override). `/fix`, `/review`,
  `/triage`, and `/validate` may be started from the REPL or from CLI
  `--prompt`; both entry points must derive behavior from the same JSON
  workflow.
- `/fix`, `/triage`, and `/validate` use the workflow executor directly.
  `/review` uses `review.json` to define its prompt contract and lenses, then
  runs through the REPL task/todo loop so followups become prioritized
  next-turn todos. Do not add a second review engine or a markdown
  prompt fallback.
- Do not reintroduce markdown prompt-template fallbacks, special REPL
  task paths, or command-specific alternate engines for workflow-owned
  commands. If workflow behavior is wrong, fix the JSON workflow and
  shared workflow runner/executor. Delete stale templates and tests
  instead of preserving alternate behavior.
- `/review` specifically must remain one JSON workflow with the old
  forward-progress semantics: each turn runs parallel slow-agent
  lenses, emits Findings plus typed followups, dedups those followups
  through the todo agent and ranks the runnable ones through the
  prioritization agent into the next review task, and continues until
  the turn cap or followup exhaustion. Do not replace
  this with workflow-local "fetch followups and repeat the same step"
  logic.
- Do not weaken the golden review prompt contract: every lens is
  exhaustive for its bug class, does not stop after the first issue, and
  emits typed followups when more source, callers, history, or API
  context is needed to be confident. A clean lens means confident, not
  merely "nothing proved from the first gathered context."
- For commit/range reviews, audit the semantic contract changed by the
  diff, not just the edited lines. If a patch changes representation,
  helper family, callback/dispatch, allocation, lifetime, locking,
  ordering, or accounting behavior, trace unchanged readers, writers,
  callers, callees, callbacks, setup/registration sites, and shared
  helpers that still rely on the old contract. Do this generically by
  following the changed contract; do not hardcode subsystem-specific
  rules. Missing unchanged paths are followups, not a clean review.
- A workflow step's synthesis call must run under a system prompt that
  matches the step's declared outputs. `agent: fast` defaults to
  `workflow-synthesis`; `agent: slow`/`code` default to the per-mode
  slow prompt; a step may name another with `synthesis_system`. Do not
  route a schema-bearing synthesis call through the fast-gather prompt:
  that prompt mandates the `ready_for_slow` envelope, and measured over
  384 validate runs the model obeyed it instead of the schema on 397 of
  784 calls, each rejection re-running the whole step.
- A workflow step's call-invariant instruction text (skills, includes)
  belongs in the prompt envelope's `stable_instructions` field, which
  gets its own cache block, not concatenated onto `question`. Nothing
  task-specific may enter that field: a head that varies per call is
  written once per call and read never, which costs more than not
  caching it.
- Reasoning invariants that quantify over a step's typed arrays, or
  that read across steps, belong in a `builtin` eval, not in prompt
  text and not in a `field_check` (the expression language has no
  quantifier). Validate's two — `validate_claims_wellformed` and
  `validate_verdict_consistency` — exist because the prompt already
  stated the rule and runs talked themselves out of it by relabelling a
  load-bearing unresolved claim as a severity question.
- Parallel runs never share a directory. Every kres run owns its
  artifacts — findings, report, session state, workflow snapshots —
  under `--results` when given and `~/.kres/sessions/<ts>-<pid>/`
  otherwise. Two runs are working on different inputs, so a file they

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masoncl/kres](https://github.com/masoncl/kres) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
