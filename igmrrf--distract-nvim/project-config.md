---
trigger: always_on
description: - **Feature Locked:** No adding features outside the scope of documented features. Only improvements (performance, memory, bug fixes, reliability, testing) are permitted.
---

# Repository Coding Standards

- **Feature Locked:** No adding features outside the scope of documented features. Only improvements (performance, memory, bug fixes, reliability, testing) are permitted.
- **Zero Explanatory Comments:** Write self-documenting code.
- **Fail Fast & Explicitly:** Never swallow errors or use empty catches.
- **Strict Size Caps:** File <= 400 lines, Component/Struct <= 150 lines, Function <= 60 lines.
- **Refactoring:** Zero behavior/contract changes without characterization tests.

# Universal Coding Rules & Engineering Standards

- **Feature lock is absolute.** Core features are sealed to the documented contracts. New behaviors belong in external plugins.
- **No explanatory comments.** Write code that reads on its own.
- **No backwards-compatibility shims in application logic.** Migrations, versioned schemas, and routing layers handle compatibility—not runtime `if`-branches.
- **Fail fast and explicitly.** Never silently swallow errors or fall back to ambiguous default values.

---

## 0. How to Read These Standards

- **This document is language-agnostic and applies to every project.** Language guides (`rust/`, `go/`, `typescript/`, `python/`, `lua/`) add idioms, toolchains, and carve-outs. They never relax a universal rule; where a language cannot honour one, the language guide states the exception explicitly.
- **Rule scope tags.** Rules apply everywhere unless tagged:
  - `[service]` — applies only to network-facing services (HTTP, gRPC, queue consumers, RPC).
  - `[app]` — applies only to deployed applications, not to reusable libraries.
  - `[lib]` — applies only to libraries and SDKs consumed by third parties.
- **Non-negotiable vs. tunable.** Rules about correctness, safety, and failure handling are non-negotiable. Numeric thresholds (size caps, nesting depth, parameter counts) and named tools are project-configurable defaults: a project may raise or lower them once, in writing, repository-wide — never per file, never ad hoc.
- **Tool names are defaults, not requirements.** Where a specific formatter, linter, or library is named, the requirement is *"exactly one, configured repository-wide, enforced in CI"*. The named tool is the default choice when the project has no existing one.
- **Adopting into an existing repository.** Apply to changed files first (see the lint ratchet in §1). A standard that would require a repo-wide rewrite to land a one-line fix is being applied wrongly.

---

## 1. Automated Formatting & Linting

- **The automated formatter is the sole authority.** Never format code by hand or manually organize imports. Run the repository's configured formatter before committing.
- **Configuration is repository-wide.** Formatter and linter configurations are uniform across the project and are never overridden on a per-file basis.
- **Lint is a ratchet, not an ad-hoc cleanup task.** Whole-repo errors are a hard CI gate (0 allowed). Warnings on untouched legacy files may exist temporarily, but any modified file must be 100% clean of both errors and warnings.
- **Never enforce an unratcheted zero-warning gate to force unrelated cleanup.** Blanket cleanups block focused PRs. The changed-files ratchet prevents new technical debt.
- **Autofix only what is mechanically safe.** Scrutinize automated autofixes; never apply unsafe autofixes that alter types, contracts, or runtime behavior.
- **Delete dead code manually.** Silencing a warning by renaming an unused variable to an ignored prefix retains dead code. Delete unused bindings completely.

---

## 2. Naming & Ubiquitous Language

Names are the primary documentation. Precise naming eliminates the need for inline comments.

- **Say what the thing is.** Avoid cryptic abbreviations that a newcomer would not immediately understand (`countryCode` not `cc`, `beneficiary` not `bnf`).
- **No single-letter names.** Variables, parameters, callback arguments, loop variables, catch bindings, and generic type parameters must have descriptive names (`record` not `r`, `error` not `e`, `index` not `i`).
  - This is a deliberate deviation from the terse-identifier habits of several languages. Each language guide lists the **only** permitted exceptions — conventional idioms whose meaning is universal and which linters expect (`ctx`, `t *testing.T`, Go method receivers, mathematical coordinates in a formula). Anything not on that list is a violation.
- **Standard casing follows the language's own convention, not a cross-language one.** Use the casing the language's formatter and standard library use; do not import another language's style:
  - Values, variables, functions: `camelCase` or `snake_case` per language standard.
  - Types, structs, interfaces, classes, traits, enums: `PascalCase` (or the language's equivalent).
  - Module-level constants and static globals: `SCREAMING_SNAKE_CASE` where the language has the concept.
- **Booleans read as predicates:** `isActive`, `hasPermission`, `canProceed`, `shouldRetry`. Never a bare noun (`active`) or an inverted negative (`isNotReady`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igmrrf/distract.nvim](https://github.com/igmrrf/distract.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
