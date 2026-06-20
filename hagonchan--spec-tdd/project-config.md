---
trigger: always_on
description: Run a spec-driven, test-first implementation workflow for software projects. Inserts three human review gates (architecture, patterns, tests) before any implementation, plus optional architecture discovery, SPEC creation, acceptance criteria, failing tests, frozen tests, and implementation loops until verification passes.
---


# Spec-TDD Implementation Workflow (with three review gates)

You are running a strict software-delivery workflow with **three mandatory human review gates** layered on top of test-first development:

```text
understand → architecture → [GATE 1] → patterns → [GATE 2] → specify → tests → [GATE 3] → freeze → implement → verify
```

The three altitudes mirror an industry pattern: humans set direction at the top (architecture), guide the middle (patterns/abstractions), and delegate file-level code to the agent. Each gate is enforced by hooks — you cannot bypass them by editing files manually.

User request / arguments:

```text
$ARGUMENTS
```

Respond in the user's language. Keep implementation artifacts in the repo language unless the user asks otherwise.

## Non-negotiable rules

1. **No production implementation before red tests AND all three gates approved.** You may create or repair test infrastructure first, but do not implement feature behavior until at least one relevant new/changed test fails for the correct reason AND `state.approvals.{architecture,patterns,tests}` are all set.
2. **Three review gates are mandatory.** After writing the architecture draft, after writing the patterns/abstractions draft, and after writing red tests, you MUST stop and present the artifact to the user. You must not proceed until the user replies `approved <gate>` (case-insensitive). Then you run `state-update.mjs --approve <gate>` to record the approval before continuing.
3. **Each gate must include a sub-agent review.** Before asking the user to approve, invoke an independent sub-agent (`general-purpose`) that re-reads inputs in a clean context and writes a review report. Attach the report when you ask the user for approval.
4. **SPEC, patterns, and tests are sources of truth.** Implementation follows `00-ARCHITECTURE.md`, `00b-PATTERNS.md`, `01-SPEC.md`, `02-ACCEPTANCE.md`, `03-TEST_PLAN.md`, and `04-TRACEABILITY.md` under `.ai/spec-tdd/<feature-slug>/`. Contracts in `src/contracts/` (if produced) are part of the patterns artifact and may not be silently rewritten during implementation.
5. **Freeze before implementation.** After red tests are proven AND tests gate is approved, run the freeze script. Frozen tests, acceptance criteria, traceability, contracts, and verify scripts must not be edited during implementation unless the user explicitly approves.
6. **Verification owns completion.** Completion requires the configured verify command to exit 0. Do not claim completion while verify fails.
7. **Fix root causes only.** Do not delete, skip, weaken, hardcode around, or mock away behavior just to pass tests.
8. **Prefer behavior tests through public interfaces.** Public surfaces are defined in `00b-PATTERNS.md`. Avoid testing private functions, internal call order, and implementation details. Mock only uncontrollable boundaries: network, clock, filesystem, platform APIs, payment providers, push notifications, external services.
9. **Slice large work.** For large projects, produce the full architecture + patterns + SPEC + test plan first, then implement one vertical slice at a time: one behavior group red → freeze → green → refactor → extend with the next slice. Each slice's tests still need a tests-review approval if SPEC/AC entries are added.
10. **Be honest about blocked verification.** If dependencies, credentials, emulator/simulator, DB, or browser drivers are unavailable, create the best local tests possible and report the exact blocked command and missing requirement. Do not pretend verification passed.

## Supporting files bundled with this skill

Load these only when needed:

- `templates/01-SPEC.template.md` — canonical SPEC structure.
- `templates/02-ACCEPTANCE.template.md` — acceptance checklist and Definition of Done.
- `templates/03-TEST_PLAN.template.md` — test strategy and red-test protocol.
- `templates/04-TRACEABILITY.template.md` — requirement ↔ test ↔ command matrix.
- `templates/05-TASKS.template.md` — vertical-slice task breakdown.
- `templates/00b-PATTERNS.template.md` — module map + ADRs + interface contracts.
- `templates/verify.sh.template` — portable verification script template.
- `reference/test-design.md` — test-quality rubric for web, mobile, desktop, backend, CLI, and libraries.
- `reference/spec-quality-rubric.md` — SPEC quality gates.
- `reference/platform-matrix.md` — platform-specific verification examples.
- `scripts/state-update.mjs` — manages `.ai/spec-tdd/state.json` (phase, approvals, slug, verify command).
- `scripts/freeze-tests.mjs` — records hashes of frozen tests and gate files (refuses to freeze if any gate is unapproved).
- `scripts/check-frozen-tests.mjs` — fails if frozen tests/gates changed or new skip/only markers appear.
- `scripts/guard-frozen-tests.mjs` — hook that blocks direct edits to frozen files during implementation.
- `scripts/guard-review-gates.mjs` — hook that blocks edits outside the review-phase whitelist when a gate is unapproved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HagonChan/spec-tdd](https://github.com/HagonChan/spec-tdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
