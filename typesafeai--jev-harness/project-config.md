---
trigger: always_on
description: These instructions apply throughout the repository unless a more specific `AGENTS.md` appears in a subdirectory. They are written for coding agents and for humans; both are held to the same boundaries.
---

# Agent guide — jev-harness

These instructions apply throughout the repository unless a more specific `AGENTS.md` appears in a subdirectory. They are written for coding agents and for humans; both are held to the same boundaries.

Start with `README.md`, then `docs/architecture.md`, `docs/roadmap.md`, and the code and tests for the module being changed. Read before editing.

## What this project is

A coding-agent harness in which an LLM proposes one action, TypeSafe AI's Jev answers four narrow yes/no (`noul`) questions about it, and pure code turns those answers into one of four verdicts: `permit`, `proposal_only`, `reject`, `unavailable`. A host constructs and stores each receipt; a fixture-bench runner in `src/benchmark/` records receipts for synthetic fixtures only. Nothing in this repository executes a proposal.

The whole value of the project is the boundary between *evidence* and *authority*. Keep it sharp:

- Jev's answers are **evidence**.
- The decision table is **policy**, in code, tested.
- **Authorization and execution belong to the host**, never to this package.

## What this project is not

- Not an official TypeSafe AI product, SDK, or endorsed harness. It lives in the independent TypeSafeAI community organization. Do not write copy that implies otherwise.
- Not an agent runtime. There is no loop here, no tool executor, no session, no identity binding.
- Not a safety guarantee. `permit` means four answers were favorable at a threshold on one pinned model. Do not describe it as "safe", "approved", or "verified".

## Orientation

```text
src/contract/types.ts    shared types; the wire shape of Proposal, ReviewAnswer, Receipt, Fixture
src/contract/decide.ts   decide(), decideBase(), unfavorable(), FAVORABLE, REVIEW_CONFIDENCE_THRESHOLD
src/contract/index.ts    root exports; benchmark-only decideBase is not re-exported
src/contract/payload.ts  Question/RunPayload types and validateReviewPayload (criteria-preserving)
src/contract/validate.ts proposal schema/path/diff validator (zod); diff.ts is its parser
src/contract/review.ts   question set v1, buildReviewPayload, reviewProposal over an injected transport
src/benchmark/          explicit base helper, offline evaluation/blinding, fixture bench (load.ts is Node-only)
fixtures/proposal-review/ the synthetic proposal-review fixtures (20 extracted, 1 added for #4, 4 for #5)
src/audit/receipt.ts    optional Node binding/replay adapter; not a pure-root import
src/routing/            pure catalog, normalized evidence seam, routing policy, context assembly
examples/routing/       synthetic routing scenarios and paired comparison
tests/*.test.ts          node:test via tsx, offline
docs/architecture.md     the design of record; update it when behavior changes
docs/roadmap.md          phases and exit criteria; update it when a phase lands
.github/workflows/       CI: pnpm install --frozen-lockfile → typecheck → test; separate secret-scan job
.githooks/ scripts/      pre-commit secret check (dependency-free) installed by pnpm's prepare step
```

`src/contract/` was extracted from `TypeSafeAI/typesafe-playground` `lib/harness/` (branch `feat/proposal-review`, commit `245167d`). The phase 1 validator, review payload builder, mock transport, fixtures, and bench were re-diffed against canonical merged playground commit `6fe5967dc020521a0731682b06c4d8eeeab95ffb` (PR #41). Do not reimplement playground behavior here from memory; extract it so the two stay identical, except where the hardened contract deliberately differs.

## Package manager

Node.js 22+ and pnpm only. The exact pnpm version is pinned in `package.json` (`packageManager`). Use `pnpm install --frozen-lockfile`, `pnpm <script>`, and `pnpm exec <tool>`.

Do not use npm, npx, Yarn, or Bun for installation, scripts, or tool execution. `pnpm-lock.yaml` is the only lockfile. Do not change dependency versions, loosen a frozen install, or regenerate the lockfile to get unrelated work through. A dependency change is its own PR with its own reason.

## Contracts to preserve

### Verdicts

`ReviewVerdict` is exactly `"permit" | "proposal_only" | "reject" | "unavailable"`. Do not add a fifth. Do not rename one. Do not add a verdict, field, or helper whose name reads as "safe", "approved", "authorized", or "verified".

### The decision table

`decide()` is the normal review decision table. The internal `decideBase()` table is benchmark-only; consumers import its provenance-preserving wrapper from `src/benchmark`, never use it as a failure fallback.

- Malformed validation, `validation.ok !== true`, or nonempty validation errors → `reject`. The host must not call Jev first and records `jev: null`; this pure function cannot enforce prior host call order.
- Missing/malformed review envelope, `jev === null`, `jev.answers === null`, or a non-null review error → `unavailable`. The reason must say it is treated as proposal-only, never as safe.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TypeSafeAI/jev-harness](https://github.com/TypeSafeAI/jev-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
