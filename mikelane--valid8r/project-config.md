---
trigger: always_on
description: [available_instructions]
---

# .cursorrules
# Goal: Make Cursor an excellent software engineering companion. Combine general, language-agnostic engineering rules
# with repo-specific testing/style guidance already in use.

Cursor Project Rules

[available_instructions]

# ---------- Language-agnostic AI operating rules ----------
ai_alignment:
  # Clarify the ask and fit the context
  • Restate the task, inputs, outputs, constraints, and success criteria before writing code.
  • Detect and follow the project’s stack, runtime, CI, testing style, and naming/layout conventions.
  • Surface tradeoffs: propose a primary approach and at least one viable alternative with pros/cons.

ai_small_steps:
  # Keep changes tiny and reversible
  • Prefer minimal, incremental diffs gated by tests; one responsibility per change.
  • Use feature flags/toggles or adapters for risky changes.
  • Avoid speculative generality; implement the smallest thing that could possibly work.

ai_correct_first:
  # Make it work before making it fast/fancy
  • Provide runnable examples and tests that demonstrate behavior.
  • Specify behavior via public surfaces; avoid coupling tests to internals.
  • Cover golden path, edge cases, and failure modes.

ai_readability:
  # Clear beats clever
  • Favor descriptive names, short functions, cohesive modules.
  • Remove duplication; maintain a single source of truth.
  • Isolate complex logic behind simple interfaces.

ai_design_for_change:
  # Stable boundaries, evolvable internals
  • Program to interfaces/abstractions; hide implementation details.
  • Validate at the edges; keep core logic working on trusted data.
  • For legacy code, prefer strangler patterns and incremental replacement over rewrites.

ai_refactoring:
  # Improve safely with a net
  • Refactor only with passing tests; preserve behavior.
  • Apply the scout rule: leave touched code cleaner.
  • For perf-sensitive areas, capture before/after metrics.

ai_security_reliability:
  # Non-optional requirements
  • Enforce least privilege for code, data, secrets, and CI.
  • Validate and sanitize all inputs (type/range/format); fail closed with secure defaults.
  • Add observability at service/module boundaries: structured logs, key metrics, trace points.

ai_performance:
  # Pragmatic optimization
  • Make it work, then measure; optimize only with evidence.
  • Prefer simpler algorithms until data proves otherwise.
  • Call out expected time/space/IO complexity and resource budgets.

ai_dependency_hygiene:
  # Keep the supply chain tidy
  • Prefer standard libraries and fewer dependencies when reasonable.
  • Track versions, licenses, and security advisories; pin/lock where appropriate.
  • Wrap third-party libraries behind thin, swappable adapters.

ai_docs:
  # Documentation that earns its keep
  • Update API/module docs when behavior or surfaces change; describe behavior, inputs, outputs, invariants.
  • Prefer runnable examples to prose where possible.
  • Delete or update stale docs; stale docs are bugs.

ai_vcs_etiquette:
  # Version control best practices
  • Make atomic commits with messages that explain “why” before “what.”
  • Ensure tests/linters/static analysis pass locally before opening a PR/MR.
  • PR/MR descriptions should include context, decisions, risks, and validation notes.

ai_collaboration:
  # Human-friendly by default
  • Ask clarifying questions when requirements are ambiguous; do not guess silently.
  • Offer 2–3 options with consequences when tradeoffs exist.
  • Match the project’s code style, directory layout, and tooling.

ai_ethics_guardrails:
  # Do the right thing
  • Respect licenses; verify compatibility before suggesting code.
  • Attribute significant ideas/snippets that are not original.
  • Never expose secrets or PII in code, logs, examples, or test data.

ai_definition_of_done:
  # Quality bar for AI-produced changes
  • Task restated; constraints and acceptance criteria confirmed.
  • Approach and alternatives documented (in PR/MR or design note).
  • Code compiles/runs cleanly; linting/formatting/static analysis are clean.
  • Tests exist, pass locally, and demonstrate behavior and edge cases.
  • Security, error handling, and observability addressed where relevant.
  • Public surfaces documented; docs/doctests updated.

ai_escalation:
  # Know when to pause and propose a design
  • Conflicting requirements or missing success criteria.
  • Meaningful risk of data loss/security exposure without a rollback plan.
  • Changes imply architectural shifts; produce a short design note first.

ai_operating_sequence:
  # Step-by-step workflow for Cursor
  1. Confirm: restate task/constraints/acceptance criteria; ask blocking questions.
  2. Plan: outline a tiny increment that delivers user value; note tradeoffs.
  3. Propose: show tests first, then implementation; keep diff small.
  4. Validate: enumerate edge cases, security checks, and perf notes.
  5. Refine: simplify names/structure; remove duplication.
  6. Deliver: provide patch + tests + concise PR description with rationale and follow-ups.
  7. Iterate: accept feedback; repeat in small steps.

# ---------- Existing repo-specific testing/style guidance ----------
pytest_naming:
  Use pytest discovery configuration from pyproject.toml:
    • test paths: tests
    • test files: it_*.py, test_*.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikelane/valid8r](https://github.com/mikelane/valid8r) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
