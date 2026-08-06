---
trigger: always_on
description: > **Top rule:** Never accept a healthier user-facing state than the persisted
---

# AgentSpellbook Development Agent Instructions

> **Top rule:** Never accept a healthier user-facing state than the persisted
> evidence supports. When uncertain or partially failed, report
> degraded/stale rather than fresh/complete.

> **Compliance rule:** When existing code, tests, documentation, or checklists
> violate this file, correcting that violation takes priority over starting the
> next milestone. Adding a rule does not resolve the defect that motivated it.

## Project context

Repository: `$REPO` (the local AgentSpellbook checkout)
Rust workspace, edition 2024, toolchain pinned at 1.96.1.
Hexagonal architecture: domain <- application <- transports (CLI/MCP).
Application crate must not depend on SQLite, Clap, MCP, tokio, regex, or
platform crates.

## How to read current state

`AGENTS.md` intentionally avoids hardcoded HEAD SHAs, test counts, and
milestone completion status because they expire between sessions. Read the
live state with commands instead:

```bash
git log --oneline -5                                  # recent commits
git rev-parse HEAD                                    # current commit
cargo test --workspace --all-targets 2>&1 | grep 'test result:'  # test count
git status --short                                    # working tree
```

Dated snapshots (commit SHA, toolchain, schema version, test count,
release-binary hash, provider fingerprints) live in
[docs/release/m0-baseline-capture.md](docs/release/m0-baseline-capture.md).
Milestone task status lives in
[docs/release/m0-release-checklist.md](docs/release/m0-release-checklist.md)
and is the single source of truth for what is done.

### Trial readiness plan

7 engineering days (M0-M5) + 14-day user trial. The full plan with complete
task IDs, exit gates, and trial success criteria lives in
[docs/release/user-trial-readiness-plan.md](docs/release/user-trial-readiness-plan.md).

- **Feature freeze in effect (M0-05):** no new feature work is accepted until
  M5 is complete. Only milestone tasks in the trial readiness plan proceed.
  Any out-of-scope request requires a separate product decision.
- Check the [release checklist](docs/release/m0-release-checklist.md) for
  current milestone status rather than relying on this file.

## Mandatory defect-closure protocol

Passing the existing test suite is not the same as passing acceptance.

A correctness, persistence, privacy, freshness, cursor, migration, or transport
defect is closed only when all five closure artifacts exist:

1. **Production fix**
   - The root cause is fixed, not only the reviewed line or visible symptom.

2. **Same-root audit**
   - Search the affected data flow for the same unsafe pattern.
   - Inspect inputs, persistence, public response, CLI, MCP, restart behavior,
     and aggregate state where applicable.
   - Classify every match as fixed, explicitly justified, or deferred with an
     owner and milestone.

3. **Negative regression test**
   - Reproduce the pre-fix failure.
   - Exercise the real public binary or real MCP JSON-RPC transport.
   - Inject the failure rather than manually writing the expected final state.
   - Assert structured JSON fields, exit code, persisted state, and behavior
     after process restart.
   - Privacy-sensitive changes must use secret, Unix-path, Unicode-path,
     Windows-path, and UNC canaries.

4. **Claim reconciliation**
   - Update checklist items, test names, evidence documents, and milestone
     status to describe what the tests actually execute.
   - A test that writes state through the Store API is not a real sync test.
   - A binary used only for retrieval does not prove that the binary performed
     the preceding state transition.

5. **Committed evidence**
   - The implementation, tests, and updated claims must all be tracked and
     committed before reporting completion.

If any closure artifact is missing, report `IN PROGRESS`.

### Outcome-first execution

Before editing, write one outcome sentence:

> Make [user-visible state] truthful across [real operation] and [restart or
> transport boundary] under both success and injected failure.

Then define:

- in-scope files and surfaces;
- failure scenarios;
- required public-path evidence;
- explicit non-goals;
- COMPLETE versus IN PROGRESS conditions.

Do not begin implementation until this outcome contract is written.

### Full-path audit

For runtime state and diagnostic changes, inspect the complete path:

```
source/provider
-> discovery/probe/ingestion
-> cursor and state persistence
-> aggregation
-> public response
-> CLI
-> MCP
-> process restart
```

A fix at one stage is incomplete if a later stage can leak, suppress,
contradict, or fabricate the same information.

### Correctness-change test rule

A production change involving persistence, privacy, freshness, cursor,
migration, state transition, or error propagation must add or materially
strengthen a regression test for the failing path.

An unchanged passing test count is not evidence that the newly changed failure
path is covered. If an existing test already covers it, the completion report
must identify the exact assertions that failed before the fix and pass after
it.

### No literal-finding fixes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenyujian/agentspellbook](https://github.com/chenyujian/agentspellbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
