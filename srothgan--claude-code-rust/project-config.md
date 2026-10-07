---
trigger: always_on
description: - Be precise, concise, and evidence-based. Avoid vague statements and unnecessary long explanations.
---

# AGENTS.md

## Working Style

- Be precise, concise, and evidence-based. Avoid vague statements and unnecessary long explanations.
- Inspect the relevant code, configuration, tests, and existing patterns before changing behavior.
- For clear, scoped implementation requests, proceed without asking for another approval.
- For open-ended feature or design requests with more than one viable approach, present the options and a recommendation before editing.
- Ask for clarification only when ambiguity materially affects behavior, scope, data, security, or an irreversible/external action.
- Verify assumptions that could affect correctness. When version- or API-specific behavior matters, prefer official documentation.
- Keep changes focused on the requested work. Avoid unrelated refactors or cleanup.
- Prefer established repository patterns unless there is a concrete reason to change them.

## Discussing

Focus on substance over praise. Skip unnecessary compliments, engage critically with my ideas, question my assumptions, identify my biases, and offer counterpoints when relevant. Don't shy away from disagreement, and ensure that any agreements are grounded in reason and evidence.

## Change Discipline

### Complete the relevant change

Understand which parts of the system the requested behavior actually crosses and update all affected parts.

Depending on the task, this may include runtime behavior, domain logic, persistence, schemas, generated artifacts, external interfaces, tests, documentation, or obsolete code.

Do not treat this as a mechanical checklist. Update what is relevant to the real workflow and no more.

### Follow the real workflow

Reason from the actual entry point through downstream behavior instead of modifying only the directly mentioned function or file.

Where relevant, consider:

- success and empty cases;
- partial and failure paths;
- retry or reuse behavior;
- boundary conditions;
- persistence or serialization round-trips;
- downstream consumers.

Do not add handling for hypothetical states that are not reachable through the supported workflow unless explicitly requested.

## Architecture

### One authority per invariant

Each important validation, invariant, transition, relationship, decision, or derived value should have one authoritative owner.

Do not duplicate the same semantic rule across callers and callees, services and domain objects, persistence and runtime models, generated and handwritten models, or multiple helpers.

Boundary validation may validate a relationship or input at the boundary, but should not reimplement invariants already guaranteed by the authoritative object.

### Avoid parallel or driftable state

Do not introduce unnecessary:

- copied semantic fields;
- mirrored models;
- duplicated derived values;
- alternative decision paths;
- compatibility paths that keep two implementations active;
- multiple implementations of the same calculation or rule.

Representations at different layers may differ in shape, but they should agree on the same semantics and source of truth.

### Fix root causes

When a defect comes from incorrect ownership, duplicated state, an inconsistent invariant, or a broken workflow, fix that cause rather than layering guards, filters, fallbacks, validators, or special cases around the symptom.

Additional defensive checks are appropriate when they protect a real boundary or failure mode, not when they compensate for an unresolved architectural problem.

## Replacement and Cleanup

When behavior is replaced, remove the obsolete implementation instead of leaving parallel architectures active.

Check for stale:

- validation;
- helpers;
- branches;
- fields or enums;
- imports or parameters;
- comments or diagnostics;
- tests;
- compatibility paths.

Do not remove an important behavioral guarantee merely because its previous implementation was replaced. Move the guarantee and its regression coverage to the new authoritative owner.

Keep cleanup scoped to code made obsolete by the requested change unless broader cleanup is explicitly requested.

## Testing and Verification

- Add regression coverage for changed behavior when appropriate.
- Prefer tests of the authoritative behavior and real workflow over tests that only exercise isolated helpers.
- Define each test by the intended user or domain guarantee and the concrete defect it must catch. A passing test must distinguish the required behavior from a plausible broken implementation.
- Derive expected results from requirements or independent fixtures, not from the production helper or calculation being tested.
- Assert observable outcomes at the relevant boundary: persisted data, emitted commands, delivered notifications, interaction responses, or the relevant rendered region. Check the actual target and seed the state named by the test.
- Use exact string assertions when the string itself is required output, such as copied content, time formatting, protocol bytes, or explicitly required UI wording. Do not lock down incidental prose, punctuation, or complete footer text.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srothgan/claude-code-rust](https://github.com/srothgan/claude-code-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
