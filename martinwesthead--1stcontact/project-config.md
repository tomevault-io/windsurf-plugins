---
trigger: always_on
description: provides: [batch_index, ...]
---

# Claude Instructions for 1stcontact

**Note:** This project uses XGD (Extreme Generative Development) methodology. XGD methodology docs are bundled in the xgd package (`xgd_source/system_docs/`) and loaded automatically into sessions.

## XGD Workflow Documentation

**System Documentation** (bundled in xgd package — `xgd_source/system_docs/` — read-only):
- `TDD-PROCESS.md` - TDD RED/GREEN workflow, quality gates, exception policy
- `TEST-STRATEGY.md` - Testing guidelines, what to test, parameterization
- `TASK-TEMPLATE.md` - Template for creating streamlined task prompts

These docs are bundled in the xgd package and loaded at runtime — not copied into your project.

## Branch Topology

**See `docs/reference/xgd-branching-strategy-full.md`** for the authoritative branch topology reference.

Summary:
- `xgd-working` — operator hot branch; all free-coded changes, tickets, config
- `main` — tested, reconciled truth; all branches cut from here
- `branch-<ticket>` — develop/feature branches; cut from main, merge back to main
- Reconcile branches — short-lived; promote working bundles into main
- Resync branches — short-lived; rebase working's unreconciled tail onto updated main
- Regression branches — cut from main; on success, fast-forward `xgd-stable`
- Merge branches — mechanical plumbing; created and deleted within a single run
- `xgd-stable` — production release; guaranteed conforming matrix and passing tests

## ⚠️ CRITICAL: No Legacy Modes or Backward Compatibility

**See also**: `PHILOSOPHY.md` (xgd system docs) → "Simplicity Over Preservation" and "Ruthless Refactoring"

**When refactoring, replace fully. Do NOT create legacy/fallback modes unless explicitly requested.**

- When implementing a new approach, replace the old one completely
- Do NOT add "legacy mode" or "fallback to old behavior" code paths
- Do NOT auto-detect "which mode to use" between old and new implementations
- Delete old code; don't comment it out or hide it behind flags
- Tests validate correctness, not legacy code paths
- Git history is the archive if old code is ever needed

**Why**: Legacy modes create:
- Complexity (two code paths to maintain)
- Confusion (which mode am I in?)
- Bugs (mode detection logic, edge cases)
- Dead code that never gets removed

**The Rule**: When replacing an approach, delete the old one. If tests pass, the new implementation is correct.

## ⚠️ CRITICAL: Failure vs Error Taxonomy (Workflow Outcomes)

**Failures, terminal failures, and errors are fundamentally different. Do NOT confuse them.**

| Category | What it is | Expected? | Recoverable? | System behavior |
|----------|-----------|-----------|--------------|-----------------|
| **Failure** | Tests fail, review fails, quality check fails | Yes | Yes (fix loop) | Retry via fix workflow (`@fail` → `fix_*` state) |
| **Terminal failure** | Guard not met, spec insufficient, validation gate | Yes | No | Graceful halt, clear message (`on_fail: terminal_failure`) |
| **Error** | Bug in system: missing FSM transition, broken config | **No** | **No** | **Immediate termination**, no handling, no cascading |

### The Distinction That Matters

- A **failure** is a workflow producing a negative result. The system has a defined path for it (fix loop).
- A **terminal failure** is an expected dead-end. The system stops gracefully.
- An **error** is a bug. The system is broken. There is no defined path because one should have existed but doesn't.

### Errors Must Not Be Handled As Failures

When the system encounters a bug (e.g., missing FSM transition), it MUST:
- **Terminate immediately** — do not cascade `@fail` through container hierarchy
- **Print a single clear error message** — not 3 cascading "failure" messages
- **Use distinct messaging** — "ERROR (BUG)" not "workflow failed"
- **NOT attempt recovery** — no transition, no retry, no fallback

### Tag Suffixes

- `@done` — workflow completed successfully
- `@pass` — gate/check passed
- `@fail` — workflow produced negative result (recoverable)
- `@loop` — iteration continues
- `@skipped` — skip condition met, workflow not needed
- `@error` — system bug, immediate termination (NOT a workflow outcome)

## ⚠️ CRITICAL: Prerequisite Semantics — Preconditions, NOT Flow Control

**Prerequisites are system preconditions. They are NOT workflow logic or flow control.**

A prerequisite checks that the system is in a valid state to run the workflow. If a prerequisite fails, it means there is a **bug** — missing data, broken config, corrupt state. The system MUST terminate immediately.

**Prerequisites MUST NOT:**
- Return `status: "done"` to signal "nothing to do" — that's flow control (use `skip_if` instead)
- Return `success: false` as a soft signal — it causes `FatalWorkflowError` → immediate termination
- Contain business logic about whether work remains — that belongs in `skip_if` or `exit_conditions`
- Be used to decide loop continuation — that's what `exit_conditions` with `loop`/`done` suffixes are for

**Prerequisites MUST:**
- Validate that required data exists (tickets, reports, config)
- Populate context variables needed by the workflow action
- Raise/fail immediately if the system state is invalid

**Anti-pattern (causes thrashing loops):**
```yaml
# ❌ WRONG: Prerequisite returns status="done" when no batches remain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinwesthead/1stcontact](https://github.com/martinwesthead/1stcontact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
