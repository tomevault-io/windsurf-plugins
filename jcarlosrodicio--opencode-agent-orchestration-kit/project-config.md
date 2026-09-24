---
trigger: always_on
description: enables scheduling, parallelism, network, write MCP, worktrees, commit, push,
---

# Agent Contracts

`docs/ai/harness/orchestration-contracts.json` declares the portable agent
inventory, mode, semantic write authority, and delegation relationships that
the checker can compare mechanically. Agent frontmatter remains the effective
OpenCode configuration; this document retains explanations, exceptions, and
expectations outside the schema. Version 1 validates both surfaces but does not
generate or modify frontmatter.

## Usage Matrix

| Agent | Use when | Do not use when | Expected evidence |
| --- | --- | --- | --- |
| `developer` | Change is small, clear, approved, or delegated by lead | Critical decision, design, or research is missing | Validation run or explicit reason |
| `lead` | Free-form message without slash command, slash workflow, coordination, or phase dependencies | The task was already delegated to another agent | Routing decision, assumptions, criteria, and barriers respected |
| `designer` | UX/UI, brand, layout, interaction, or Open Design matters | Technical change has no visual impact | Visual handoff with observable criteria |
| `researcher` | Technical/product uncertainty, APIs, libraries, risks | Fact is already clear in repo | Sources reviewed and unknowns remaining |
| `specifier` | There is enough context to turn goal into tasks | Critical research/design is pending | Acceptance criteria and validation plan |
| `reviewer` | A diff, implementation, or `/plan` artifact exists | There is no reviewable change or planning artifact | Final canonical verdict with causal findings and evidence |
| `review_coordinator` | `/review-orchestrated` is invoked; runs as a primary session | A final integral verdict is required | Preflight/partial stage, reviewer states, and deduplicated findings without a final verdict |
| `review_quality` | General correctness or maintainability is selected | Change is skipped or has no relevant patch | Structured quality findings or `[]` |
| `review_security` | Auth, permissions, secrets, infrastructure, dependencies, or migrations are risky | No security signal exists | Structured security findings or `[]` |
| `review_tests` | Tests, validation, or regression risk is selected | Documentation/generated-only change has no risk | Structured validation findings or `[]` |
| `review_api` | APIs, schemas, contracts, CLIs, or configuration change | No public contract surface changes | Structured compatibility findings or `[]` |
| `scoper` | User wants research -> spec without implementation | User asks for direct implementation | Scoped spec and ordered tasks |
| `evaluator` | Benchmark/smoke evidence or `/evolve` is needed | Normal feature already has clear validation | pass/fail/not_run results |
| `debugger` | Failures, traces, results, or attribution need analysis | There is no concrete evidence | Root cause or not-ready state |
| `evolver` | Harness improvement has AHE evidence | Normal app feature | Manifest with predicted fixes and risk tasks |

## Invariants

- `lead` is the harness `default_agent` and acts as a bounded router for free-form messages.
- `lead` must not force the full flow for small free-form messages.
- `lead` does not edit files; if implementation or correction requires repo
  changes, delegate to `developer`.
- `lead` does not develop, deeply investigate code, or review diffs as a
  substitute for `researcher` or `reviewer`; it only gathers minimum routing
  context.
- If `lead` needs to understand how the code works before deciding what to do,
  it delegates substantive discovery to `researcher`.
- **`lead` always delegates and never implements**, whatever the size of the
  change. The rule was already written down, but it was not enforced: with
  `--auto` an `ask` permission is auto-approved, so `bash: "*": ask` let `lead`
  run anything, writes included. Measured: `lead` implemented on its own with
  grep/glob/read/bash instead of delegating. It is now `"*": deny` with a
  read-only allowlist, plus the exact scripts its own commands tell it to run
  (`loop-state.mjs inspect`, `preflight-audit.mjs`). A shell plugin that
  rewrites commands must have its rewritten forms allowlisted too, or they fall
  through to the deny.
- `lead` emits its routing decision as **fields, not prose**: `route`,
  `confidence`, `skipped` and `why` (one line, only when `confidence` is `low`).
  It does not justify the agents it discards. Measured on the
  `feature-tag-normalizer` case, the routing turn generated between 3,776 and
  13,116 output tokens, much of it paragraphs explaining why `designer` and
  `researcher` did not apply to a twenty line function.
- If there is a diff, implementation, or reviewable plan, bug, security,
  regression, and compliance review belongs to `reviewer`.
- Every `lead` handoff to another agent must be self-contained: objective,
  minimum context, constraints, assumptions, expected output, and expected
  validation or evidence.
- During lightweight shell inspection, `lead` should prefer exact allowlisted
  primitives already named by the user, without drifting to nearby substitutes
  or compound shell commands when a single call is enough.
- `developer` executes direct mode when `lead` delegates a small, clear, verifiable task.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jcarlosrodicio/opencode-agent-orchestration-kit](https://github.com/jcarlosrodicio/opencode-agent-orchestration-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
