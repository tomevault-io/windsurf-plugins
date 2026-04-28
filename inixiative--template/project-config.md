---
trigger: always_on
description: Canonical instructions for all coding agents in this repository.
---

# AI ENTRYPOINT

Canonical instructions for all coding agents in this repository.
`AGENTS.md` and `CLAUDE.md` should symlink to this file.

## 0. Purpose and Priority

**This is a template, not a product.** Developer experience (DX) is the #1 priority. Every decision — naming, file structure, API design, decomposition — should optimize for the next developer who reads the code.

- **Naming matters.** If a name doesn't clearly describe what the thing does, rename it. Do not leave misleading or generic names for convenience.
- **One file, one concern.** Each file should have a single clear purpose. If a file has multiple exported functions that serve different consumers, split them. The cost of an extra file is near zero; the cost of hunting through a multi-purpose file is real.
- **No premature features.** Do not build things nobody is using yet unless they are foundational patterns that every app page will need. When in doubt, ask.
- **No continuous support debt.** Build things that are self-explanatory and do not require hand-holding. If a consumer needs to read the source code to understand the API, the API is wrong.
- **Adversarial reviews should run regularly.** After any significant implementation, run an adversarial review in the background against the full scope of changes. Do not wait for the user to ask. Flag issues proactively.

## 0.0 Critical Guardrails

- Do not bypass failures by editing CI rules/fixtures.
- Protected paths: `scripts/ci/rules/**`, `scripts/ci/run-ci-rules.sh`, `scripts/ci/rule-violations/**`.
- If a rule fails, fix the violating source code.
- Only change CI rules/fixtures when the user explicitly requests it.
- Prefer minimal, targeted changes over redesigns.
- Do not run git commands unless the user explicitly asks.

## 0.1. Decomposition by Default

Before undertaking any task or planning any approach, decompose first. Do not jump to implementation. Break the problem into its atomic concerns before deciding how to solve it.

**Interrogate the problem:**
- What are the distinct concerns here? (ownership, access, usage, lifecycle, rendering, etc.)
- What are the atomic steps of the operation?
- Which parts change independently? Those are separate responsibilities.
- Which parts have different lifecycles? (created at different times, deleted at different times, owned by different entities) Those are separate records.
- Who owns this data? Who controls access to it? Who consumes it, and in what context? If the answers differ, they are different models.

**The lazy instincts to resist:**
- "Just add a field" — stop. Is this the same concern as the model you're modifying, or a new concern wearing the same clothes?
- "Just store the URL" — stop. Is something else going to need to know about this reference? Then it's a binding, not a string.
- "Just copy the logic" — stop. Is this the same operation in a different context, or a genuinely different operation? Same operation = shared abstraction. Different operation = keep separate even if they look similar.
- "Just put it all in one table/function/component" — stop. Convenience now is coupling later.

Never merge distinct concerns into one place for convenience — the convenience is temporary, the coupling is permanent. When the impulse is to do the quick thing, pause and ask whether the quick thing creates a coupling that will cost more later. If the decomposed version is only marginally more work, do it. The codebase should get more decomposed over time, not less.

## 1. Mandatory Task Intake (Always)

Before editing, restate:

- `Mode`: minimal by default.
- `Scope`: exact files/modules to touch (`1-3` files unless expanded by user).
- `Goal`: one concrete outcome.
- `Non-goals`: what must not change.
- `Pattern`: local pattern/module to reuse.
- `Abstractions`: no new wrappers/types/hooks unless requested.
- `Validation`: exact commands to run.
- `Stop condition`: stop after first passing validation for scope.
- `Tooling constraints`: no git unless requested.

If critical info is missing, ask one short clarification question.

## 2. Execution Contract

1. Show a brief issue list and exact patch plan before edits.
2. Apply the smallest patch that solves the requested problem.
3. Run only agreed validations.
4. If blocked, stop and ask; do not branch into redesigns.
5. Report exact files changed and validation output.

Calibration default:

1. Restate scope/goal/non-goals.
2. Propose minimal patch plan.
3. Wait for user approval.
4. Implement approved patch only.
5. Report results.

## 3. Circuit Breaker (Stop and Ask)

Pause and request approval if any occur:

- Scope expands beyond agreed modules.
- More than `3` files needed for a minimal task.
- More than `2` failed attempts on same issue.
- New abstraction appears required but unrequested.
- Signature changes across multiple callers are needed.
- You cannot explain fix in 3 concise bullets.

When tripped, provide:

1. What changed from original scope.
2. Options `A/B/C` with tradeoffs.
3. Recommended next patch.

## 4. Pattern Discovery Before Implementation

Before writing new code (especially non-trivial tasks):

1. Find `2-3` similar modules.
2. Extract concrete local patterns (types, access, validation, errors, permissions).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inixiative) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
