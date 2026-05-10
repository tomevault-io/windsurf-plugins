---
trigger: always_on
description: Graph-based workflow enforcement for AI coding agents.
---

# Freelance

Graph-based workflow enforcement for AI coding agents.

## Development

- `npm run build` — compile TypeScript
- `npm test` — run all tests
- `npm run dev` — run in development mode

After a non-trivial batch of code changes (multiple files or a cohesive multi-step edit), run `/simplify` before reporting the task complete. Skip for single-line fixes, test-only tweaks, or pure config/docs edits.

## Design iteration

When iterating on a design (proposal, issue body, new mode/option, added column), check each addition against existing system invariants **before** drafting. If the case the addition addresses isn't expressible in the current schema or types, the addition is solving an imaginary problem — kill it at the premise, not at review. Memory invariants live in `docs/memory-intent.md` (architectural qualities + anti-patterns) and in the schemas under `src/memory/` + `src/schema/`; read both before proposing a new mode, column, or tool. Concrete example: `memory_emit` requires `sources: [min 1]` on every proposition, so there is no "non-source-aligned" proposition — a prune mode scoped to that case would be solving a case the schema makes malformed.

## Feature authoring checklist

Before declaring a feature done, walk the five dimensions — otherwise the first pass will miss adjacent surfaces and take multiple rounds to catch up. This is a prompt for Claude as much as a human reference.

- **Agent journey.** What does the LLM see on discovery (e.g. `freelance_guide <topic>`), on successful use (response shape, suggested tools, next edges), and on recovery (blocked vs. structural error, recovery hint)? Trace it end-to-end.
- **Operator journey.** Human at the shell with `freelance <verb>`. JSON output is the wire format; does it parse cleanly under `jq`? What does a debugging operator see on disk under `.freelance/`?
- **Adjacent-surface sweep.** Touch each relevant surface or consciously skip:
  - `docs/decisions.md` if the feature names a cross-file invariant.
  - `plugins/freelance/skills/freelance/SKILL.md` + `templates/skills/freelance/SKILL.md` (byte-identical — `plugin-manifest.test.ts` enforces) if the skill protocol changes.
  - CLI verb descriptions / tool descriptions if behavior changes.
  - Error messages that reference the new surface where the operator is likely to hit them.
  - Graph schema (`src/schema/`) if a new field or validation applies.
  - `freelance validate` if there's a new lint rule.
  - `freelance init` templates if a starter should demonstrate the pattern.
- **Migration.** Existing on-disk records (traversal JSON, memory.db rows), existing scripts calling the verb, existing graph yaml. Does the change read correctly against pre-feature state, or is a migration needed?
- **Observability.** How do success, failure, and ambiguous cases surface? Structured error envelope (`errorKind: "blocked" | "structural"`), exit codes, breadcrumbs on stderr — pick the right channel for each.

Not a PR template — keep it inline so it's visible during authoring, not a checkbox exercise on submit.

## WHY-comments vs decisions log

Inline WHY comments are fine when a reader scanning *this file* needs the context at the point of reading: a local invariant, a subtle workaround, a non-obvious choice the code itself can't express. When the WHY has cross-file implications — names an invariant other modules rely on, documents a tradeoff a future change elsewhere would break, or captures a decision deliberated in an issue thread — elevate to `docs/decisions.md` and optionally cross-link from the inline comment.

The test: could a contributor refactoring a sibling file violate the invariant without reading this comment? If yes, it belongs in the decisions log, not buried as a comment in one file. Narrating what the code does, referencing the current task/PR, or summarizing obvious flow is never a valid comment — delete on sight.

**After closing a batch of related issues on a shared rationale, scan whether the rationale belongs in the decisions log as a standalone entry.** Per-issue close comments are complete on their own terms but don't compound — the next contributor proposing a similar feature has no single anchor to cite. If the same posture appeared across three-plus closures (a common stop-line, a rejected anti-pattern, a named invariant), that's the signal to promote it to a decisions entry and cross-link the closures from it.

## Refactor backlog

`/simplify` surfaces findings that aren't always in-scope to fix on the current PR. Anything you decide NOT to fix — out-of-scope, pre-existing, low impact on this path — append as a one-line entry to `docs/debt.md`: `<file>:<line> — <finding> — <rationale for skipping>`. Flat list, no categories, no status column; keep append cheap. Mark entries done by deleting them. Before opening a refactor PR, scan the file for candidates.

## Releases

npm publishing is **CI-driven by GitHub Releases** (`.github/workflows/publish.yml` fires on `release: published` and runs `npm publish --provenance`). Do NOT `npm publish` from a local machine — the workflow uses an OIDC token and provenance that local publishes won't produce.

Flow for a patch release (e.g. 1.3.2 → 1.3.3):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duct-tape-and-markdown/freelance](https://github.com/duct-tape-and-markdown/freelance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
