---
trigger: always_on
description: TDD Guardian plugin for Claude Code. Enforces strict test-driven development with automated quality gates across unit, integration, e2e, and contract test lanes.
---

# tdd-guardian

TDD Guardian plugin for Claude Code. Enforces strict test-driven development with automated quality gates across unit, integration, e2e, and contract test lanes.

## Project structure

```
.claude-plugin/
  plugin.json             Plugin metadata
.github/workflows/
  verify.yml              CI — tests on Node 18/20/22/24, nlpm-check, badge freshness
hooks/
  hooks.json              Hook registration (auto-discovered by Claude Code)
agents/                   Specialized subagents for TDD workflow
  tdd-planner.md          Work item planning
  tdd-test-designer.md    Behavior-driven test design
  tdd-spec-adversary.md   Attacks the matrix before code exists (read-only)
  tdd-implementer.md      Small-batch implementation, red receipts
  tdd-coverage-auditor.md Coverage gate enforcement
  tdd-mutation-auditor.md Mutation testing (report-only)
  tdd-reviewer.md         Final code + test quality review
commands/                 11 commands; basename = command name (see Command naming)
                          init, probe, gate, status, plan, design-tests,
                          implement, audit-coverage, audit-mutation, review, workflow
  shared/                 Partials — not user-invocable
    load-config.md        Load, migrate v1→v2, validate lanes
    detect-tooling.md     CI-first detection with dry-run probes
    run-lane.md           Lane execution and outcome classification
    parse-coverage.md     9-format parsing and merging
    parse-mutation.md     Mutation report parsing
config/
  config.json             Default configuration template (schema v2)
scripts/
  ci/nl-artifacts-hash.py Content hash backing the nlpm score attestation
  tdd-guardian/
    lib/config.js         Config load, v1→v2 migration, validation
    lib/coverage.js       9-format coverage parser + union/weighted merge + critical paths
    lib/lanes.js          Lane selection, execution, state, freshness
    lib/exec.js           Command execution and outcome classification
    lib/verification.js   Red classification, spec fingerprints, separation check
    receipt.js            Red-receipt CLI (record / verify / show)
    pretool_guard.js      PreToolUse hook
    taskcompleted_gate.js TaskCompleted hook
skills/tdd-guardian/      9 skills
    policy-core/          Assertion hierarchy, spec strength, mock rules, change tax, gates
    lane-policy/          Test-level taxonomy
    tooling-catalog/      Per-language runners, coverage, mutation, property libs (SKILL.md + 9 references)
    test-matrix/          Test matrix design
    coverage-gate/        Coverage enforcement + critical paths
    mutation-gate/        Mutation testing
    review-gate/          Code + test quality + spec strength + change tax
    init/, workflow/      Setup and orchestration
tests/                    node --test suite (no dependencies)
```

## Conventions

### Lanes

A lane is one test tier with its own command, trigger, and coverage participation. Config schema v2 replaced the single `testCommand`/`coverageCommand` pair with a `lanes` array.

Triggers: `taskCompleted` (the hook *runs* these), `commit` and `push` (checked for *freshness*), `manual`. **`push` subsumes `commit`.**

Schema v1 configs keep working — `lib/config.js` migrates them in memory to a single `unit` lane, preserving the original two-command behaviour exactly. Never change that migration's semantics without a major version bump.

### Test quality enforcement

Three independent axes, all required for every test:

- **Assertion level** (`policy-core`) — how strongly the test verifies anything. Every test needs at least one Level 1-5 assertion; Level 6-7 only is a wiring test and is rejected.
- **Lane** (`lane-policy`) — at what level the behavior is verified. The deciding question: *would this test still pass if the real collaborator were broken?* If yes, it belongs one lane higher.
- **Specification level** (`policy-core`) — how much of the input space the test claims, S1 (one example) through S6 (metamorphic relation). A unit **with a law** — conservation, round-trip, idempotence, ordering, monotonicity — needs an S4-S6 case. A unit with no law records that explicitly.

Adding a spec level to the policy without a library to express it would be aspiration, so `tooling-catalog` names a property-testing library per ecosystem, and says **none in wide use** where that is the truth.

### Critical paths

A lane says how expensive a suite is to **run**. `criticalPaths` says how expensive the code is to get **wrong**. One repo-wide threshold cannot express both, and a repo forced to pick one number picks the low one.

Evaluated from the merged report's per-file entries. The fail-loud rules are load-bearing: a looser-than-global threshold warns; brace expansion and non-numeric thresholds are rejected rather than coerced; and three things **fail** rather than passing quietly — a report without per-file data, a glob matching zero files, and a threshold on a dimension the merge could not compute exactly. The last two were warnings until an audit pointed out that a rule enforcing nothing reading as a pass is invariant 1 wearing a different hat.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaolai/tdd-guardian-for-claude](https://github.com/xiaolai/tdd-guardian-for-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
