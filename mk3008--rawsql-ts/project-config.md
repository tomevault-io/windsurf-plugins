---
trigger: always_on
description: This file defines repository-wide rules for rawsql-ts developers.
---

# Repository Scope

This file defines repository-wide rules for rawsql-ts developers.
Deeper `AGENTS.md` files take precedence when they add stricter or narrower rules without weakening completion criteria.

## Global Guardrails

- Keep generated artifacts, fixtures, and derived docs aligned with their source assets.
- Keep scaffold code, scaffold-facing docs, and published-package smoke checks aligned when they describe the same workflow.
- Do not weaken completion criteria or skip required verification.
- Prefer `pnpm` and scoped commands when working in a package.
- Keep repository artifacts in English unless a deeper rule says otherwise.
- Keep assistant-user conversation in Japanese in this repository.
- Do not mix customer-facing guidance into this repository policy.

## Guidance Routing

- Use the repo-local guidance under `.codex/agents/` and `.agents/skills/` for planning, verification, review, and reporting details.
- Root `AGENTS.md` defines repository-wide policy only; detailed output formats and workflows belong to subagent or skill guidance.
- Before substantial multi-step work, read the relevant guidance under `.codex/agents/` or `.agents/skills/` instead of relying on root policy alone.
- For package-level Scope, Test Policy, Authority Model, Technology Policy, review-plan, or generated review view changes, use `.agents/skills/package-spec-review/SKILL.md`.
- For structured metadata migrations or rule registry changes, use `.agents/skills/structured-metadata-migration-review/SKILL.md` to check schema versioning, canonical enum parity, real fixture parsing, and evidence/display-label integrity.
- For broad generated or derived diffs that may exceed review-tool limits, use `.agents/skills/broad-generated-diff-review-packet/SKILL.md` to prepare scoped review packets before PR handoff.
- Do not turn `AGENTS.md` into the storage location for starter walkthroughs, AI onboarding prompts, dogfooding playbooks, or investigation scripts; keep those in dedicated docs or skills.

## Documentation Guardrails

- Treat README and other human-facing repository docs as reader-facing entry documentation, not AI-facing operational notes.
- Keep human-facing docs scannable: prefer short headings, short paragraphs, short sentences, and strong structure.
- Prefer separation over deletion: if content is too detailed for README, move it to linked docs instead of silently dropping important information.
- Keep repository facts, commands, contracts, and file layout accurate and easy to verify.
- Keep structured metadata sources, schema files, implementation allowlists, fixtures, and generated review views aligned when they describe the same review harness.
- Broad generated docs or API diffs should preserve source-to-generated traceability so reviewers can inspect the source decision separately from deterministic output.

### README Mode Rules

- Each substantial README section must have one primary mode:
  - tutorial
  - how-to
  - reference
  - explanation
- Do not mix modes within the same section unless the boundary is explicit.

### Mode Expectations

- tutorial
  - learning-oriented
  - optimize for first success
  - use small steps
  - minimize explanation
  - avoid alternatives unless essential for success
  - link out for deeper background

- how-to
  - goal-oriented
  - solve one concrete task or problem
  - include only the steps and decisions needed for that goal
  - avoid broad conceptual teaching

- reference
  - information-oriented
  - describe facts, commands, options, file layout, contracts, and limits
  - stay concise, structured, and neutral
  - do not add persuasion, narrative, or long explanation

- explanation
  - understanding-oriented
  - describe why, tradeoffs, design intent, constraints, and alternatives
  - do not turn explanation into a procedural guide

### README Defaults for This Repository

- README should lead with what the project is, why it exists, and how to start.
- Quickstart sections should stay short and copyable.
- Important concepts must remain available either in README or in clearly linked follow-up docs.
- Do not remove conceptual sections, follow-up reading, or navigation aids merely to shorten the README.
- When revising README, confirm that a new reader can understand the project and reach a successful first step from the first screen.

## Plan and Reporting Minimums

- When an issue, PR comment, or request includes a proposed solution, planning must still make the underlying objective explicit and check whether the proposed solution is only one tactic rather than the actual goal.
- If there is meaningful uncertainty about the true objective, decision driver, or success condition, resolve that recognition gap during planning before implementation begins.
- Plans must state the source issue or request, acceptance items, verification methods, and explicit out-of-scope items when scope is limited.
- Multi-step tasks must keep a working ledger in `tmp/PLAN.md` unless a deeper `AGENTS.md` says otherwise.
- `tmp/PLAN.md` should be updated when the plan changes, when a blocker is discovered, and when a verification or dogfooding result materially changes the current understanding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mk3008/rawsql-ts](https://github.com/mk3008/rawsql-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
