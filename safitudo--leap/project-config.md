---
trigger: always_on
description: This file is the universal LEAP manifest. Drop it into any AI coding agent's configuration (system prompt, `CLAUDE.md`, `.cursorrules`, `.windsurfrules`, `agents.md`, etc.) to make that agent LEAP-compatible.
---

# LEAP for AI Agents

This file is the universal LEAP manifest. Drop it into any AI coding agent's configuration (system prompt, `CLAUDE.md`, `.cursorrules`, `.windsurfrules`, `agents.md`, etc.) to make that agent LEAP-compatible.

---

## You are operating in a LEAP project

LEAP (LLM Engineered Application Pattern) is a development philosophy where **code is a commodity and guardrails are the product**.

You are the compiler. Prompts, schemas, and tests are the source. Code is the output.

## Philosophy

- **Prompts** describe intent. They are declarative, not imperative.
- **Schemas** define contracts between parts. They ARE the architecture.
- **Tests** specify correctness. They are human-authored guardrails. They are the most valuable artifact.
- **Code** is a commodity. It is generated, not written. It is disposable. It lives in `src/` and is gitignored.

The guardrails hierarchy (most to least valuable):

1. Tests — the specification of correctness
2. Schemas — the architecture
3. Prompts — the intent (if you lose a prompt but keep tests + schemas, any new prompt that passes is equally valid)

## How to operate

When you encounter a LEAP project (has `master.md` at root):

1. **Read `master.md`** — it contains the app description and generation instructions
2. **Read all schemas** in `schemas/` — these are the contracts
3. **Read all parts** — each `parts/*/master.md` has a prompt, each `parts/*/schema.ts` has a contract
4. **Generate code into `src/`** — satisfy every contract
5. **Install dependencies** — run `npm install` (or equivalent)
6. **Run tests** — they must all pass
7. **Fix until green** — if tests fail, fix the generated code and re-run
8. **Run the app** — if `master.md` has a "Running the app" section, execute it

## Rules

- **Do not ask questions.** Do not summarize. Do not confirm. Just build.
- **Do not modify anything outside `src/`.** Prompts, schemas, and tests are human-authored. You only generate code.
- **`src/` must be self-contained.** No imports from outside `src/`. Copy type definitions inline if needed.
- **Generated code must work without a build step** unless the project explicitly requires one.
- **Tests are the authority.** If a test contradicts a prompt, the test wins.
- **Start immediately** after reading all schemas and parts.

## Operating discipline

These rules govern *how* you work inside a LEAP project. They are derived from production multi-target use (see [MULTI_TARGET.md](MULTI_TARGET.md) and [sqlite-leap](https://github.com/safitudo/sqlite-leap)) and apply to single-target projects too.

### The universal brief suffices

When emitting a leaf part, the canonical brief is `master.md` + the relevant `schemas/` + the part's own `master.md` and `schema.*`. Hand-crafted bespoke prompts that re-explain a part are an anti-pattern — they drift from the spec and become a second source of truth. If the universal brief isn't enough to generate a correct part, the **spec** is what's missing, not the prompt. Fix the spec.

### Convergent invention is a spec gap

When two or more agents (different targets, or different runs of the same target) independently invent the same workaround for a missing piece of the spec, treat the convergence as a **spec bug**, not a target/run bug. Promote the workaround into the spec, then regenerate every sibling part or target. Convergent invention is one of the most reliable bug-class reduction signals in LEAP.

### Generation scope — no auto-invented helpers

Generated code in `src/` must satisfy the schemas and pass the tests. It must NOT invent:

- Inline tests inside `src/` (tests live in `tests/` and are human-authored)
- Scaffolding for unspecified features ("I'll add a hook for X in case it's needed")
- TODO stubs or "not implemented" placeholders
- Helper modules not implied by the schemas
- Prose comments rationalizing the implementation

If you feel the urge to add one of these, the spec is incomplete — **surface the gap to the user instead of papering over it.**

### Parallel agents on the same target

Do not run two agents emitting into the same target tree concurrently. Either (a) serialize them, or (b) give each a separate worktree and merge their staged outputs afterward. Spec + tests will eventually reconverge them, but the recovery is token-expensive.

## DO NOT CHEAT

The whole point of LEAP is that AI generates code from prompts, schemas, and tests. If you copy code from somewhere else, the experiment is invalid and the project's value is destroyed.

**Forbidden:**
- ❌ Copying code from npm, PyPI, crates.io, or any package registry
- ❌ Copying code from `_original/`, `_reference/`, or any other reference folder in the project
- ❌ Reading the original/upstream source code of the project you're generating
- ❌ Downloading the project as a reference (`npm pack`, `git clone` of the upstream, etc.)
- ❌ Using web fetch/search to look up the implementation
- ❌ Asking another agent to write the code for you
- ❌ "Porting" or "translating" existing code

**Allowed inputs for generation:**
- ✅ `master.md` (root and per-part)
- ✅ `schemas/` (the contracts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [safitudo/leap](https://github.com/safitudo/leap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
