---
trigger: always_on
description: > **Claude Code and other Claude-family agents must read this file in
---

# CLAUDE.md — Claude-specific configuration for this repository

> **Claude Code and other Claude-family agents must read this file in
> addition to [`AGENTS.md`](./AGENTS.md).** This file imports the project-
> level agent instructions by reference and adds Claude-specific guidance.

## Inherit from AGENTS.md

All instructions in [`AGENTS.md`](./AGENTS.md) apply to Claude sessions in
this repository. Treat `AGENTS.md` as authoritative for:

* Project overview and architecture.
* Build / test / validate commands (`make …`, `python -m miccai_index …`).
* Project layout and file map.
* Core invariants (schema before render, deterministic, evidence model,
  PR-only automation, no LLM dependency, etc.).
* Conventions (Python 3.10+, type hints, YAML quoting, GitHub identity).
* What NOT to do.

If a directive in `AGENTS.md` and this file conflict, `AGENTS.md` wins.
If you want to override `AGENTS.md`, propose it via PR and update both
files together.

## Claude-specific session behavior

### Read these files first

Before doing anything else in a new Claude session, read:

1. `AGENTS.md` — the project-level agent contract.
2. `README.md` — the current state of the canonical dataset.
3. `docs/ARCHITECTURE.md` — the layered design.
4. `docs/DATA_MODEL.md` — the schema.
5. `docs/TAXONOMY.md` — how classification works.
6. `config/*.yaml` — current policy.

This is fast (~10 seconds for a single read pass) and prevents the most
common agent failures: proposing changes that contradict existing
invariants.

### Plan before code

For non-trivial changes (anything beyond a one-line fix or a small
documentation edit), Claude should:

1. State the goal and the proposed approach in 2-5 bullets before writing code.
2. Identify which invariant from `AGENTS.md` is touched (e.g. "I will add a
   new taxonomy label; this touches invariant 6 (multi-label taxonomy)").
3. Identify the tests that must exist before the change is accepted.
4. Get implicit or explicit user approval before editing files.

If the user already gave a specific instruction, skip step 4 and execute.

### Plan mode

When the user says "plan", "design", "investigate", "audit", or asks Claude
to "think through" something, do not edit files. Produce a written
analysis with:

* Goal.
* Constraints (read from `AGENTS.md` and the user's request).
* Current state (cite files / lines).
* Proposed approach (numbered steps).
* Risks and trade-offs.
* Test strategy.

Only proceed to implementation after the user accepts the plan.

### Edit mode

When the user says "fix", "implement", "build", "ship", or "go":

1. Read the relevant files first (use `read` / `grep` / `glob`).
2. Make the smallest change that satisfies the request.
3. Run the relevant tests after each meaningful change.
4. Show the diff before committing.
5. Commit only when the user asks.

### Verification gate

Before reporting "done", Claude MUST:

* Run `make test` and report the actual output (do not paraphrase).
* Run `make validate-offline` if the change touches data products or schemas.
* Run `make build-offline` and inspect `dist/` if the change touches
  rendering or data products.
* Re-read the touched files if there is any doubt about whether the edit
  applied as intended (e.g. the tool result was truncated or the diff was
  unexpected).

Never declare success without running the actual command. "I would expect
the test to pass" is not a test result.

## Common tasks — Claude recipes

### Recipe: add a taxonomy label

```
Goal:        Add a new "Federated Tumor Segmentation" task label.
Files:       config/taxonomy.yaml (add entry)
             tests/test_classification.py (add test)
Verify:      make test ; make build-offline ; grep "Federated" dist/categories.json
Risk:        low — adds one label, doesn't change existing rules.
Rollback:    revert the YAML change; the classifier is rule-driven.
```

Steps:

1. `grep -n "tasks:" config/taxonomy.yaml` to see the existing entries.
2. Read the surrounding entries to match the YAML style.
3. Add the new entry with at least one regex rule.
4. Add a test in `tests/test_classification.py` that asserts the new
   label fires on a known-positive paper and does NOT fire on a known-
   negative paper.
5. Run `make test`.
6. Run `make build-offline` against the existing `data/papers.jsonl` and
   confirm the new label appears in `dist/categories.json`.
7. Commit.

### Recipe: debug a failing classifier

1. Read the failing test output. Identify the paper's title/abstract.
2. `grep -n "<word>" config/taxonomy.yaml` to find the rules.
3. Test the regex with a quick Python REPL using the same compiled pattern
   (`miccai_index.classification._compile_rules`).
4. Adjust the rule weight or pattern; never add a rule that fires on a
   single keyword alone (see `AGENTS.md` invariant 6).
5. Re-run `make test`.

### Recipe: fix a flaky CI run

1. Read the failing job's log.
2. If the failure is `update-data.yml`, check `health.issues` in the
   summary; an unexpected coverage drop is the most likely cause.
3. If the failure is `ci.yml`, run `make test` and `make validate-offline`
   locally to reproduce.
4. Never silence a test, never `--no-verify`. Fix the underlying issue.

## What Claude should NOT do here


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ambicuity/Awesome-MICCAI-2026](https://github.com/ambicuity/Awesome-MICCAI-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
