---
trigger: always_on
description: This file is the canonical instruction file for AI-assisted development in
---

# lexdb Agent Guide

This file is the canonical instruction file for AI-assisted development in
`lexdb`.

Rules copied from shared `coding-guidelines` are adapted here. Treat this file
as self-contained; do not require agents to consult external guideline
repositories at runtime.

## Core Principles

- **Question every abstraction**: before adding a layer, file, helper, hook, or indirection, ask whether it solves a current `lexdb` problem. If the answer is hypothetical, do not add it.
- **Simplify relentlessly**: three similar lines are better than a premature abstraction. A single large file is better than several tiny files with unclear boundaries.
- **Fewer files, clearer boundaries**: split only when a file has a genuinely distinct responsibility. Never split for cosmetic reasons.
- **Prefer boring code**: a straightforward conditional chain is easier to debug than a clever dispatch hierarchy.
- **Delete dead paths instead of preserving them indefinitely**: remove unused code entirely. Do not keep stale compatibility shims unless users actively rely on them.
- **Converge UX, avoid parallel workflows**: if two commands or interaction paths do nearly the same thing, prefer one consistent model unless the distinction is genuinely valuable.
- **No side effects on load**: loading a file must not mutate user state beyond definitions and registrations required by that file's API.

## Diagnosis and Change Discipline

- **Find the root cause before changing behavior**: do not patch timing, caching, or control flow until you can name the failing layer and explain why it is responsible.
- **One failed fix narrows the hypothesis**: if the first attempted fix does not hold, reduce the hypothesis space and gather evidence. Do not stack another speculative patch on top.
- **Two failed fixes stop the patching loop**: after two failed fixes on the same issue, stop changing behavior and switch to diagnosis only.
- **Fix the right layer**: move the fix to the layer that actually owns the problem instead of compensating elsewhere.
- **Keep experiments narrow**: start new directions with the smallest slice that proves the approach is worth having. Do not expand scope before the first slice shows real value.

## Architecture

- **Dependency direction is one-way**: adapters depend on `lexdb.el`; `lexdb-ui.el` depends on `lexdb.el`; core code must not depend on adapter files; generic code must not hardcode one dictionary's schema.
- **Keep file responsibilities narrow**:
  - `lexdb.el`: core structs, registry, lookup API, shared DB/cache helpers.
  - `lexdb-ui.el`: rendering, navigation, faces, interactive UI behavior.
  - `lexdb-ldoce.el`, `lexdb-oald.el`, `lexdb-ode.el`: adapter-specific queries and data shaping.
  - `scripts/*.py`: MDX/HTML parsing and SQLite generation.
- **Schema changes are cross-cutting**: if you change the SQLite schema or attribute layout, update Python writers, Emacs readers, and `schema.md` together.
- **Capability-driven design stays generic**: if rendering or lookup is optional per dictionary, model it as a capability or adapter hook instead of branching generic code around one source.
- **Reuse Emacs infrastructure**: prefer `completing-read`, `special-mode`, text properties, standard hooks, and built-in navigation facilities over custom mini-frameworks.

## Naming

- Public Elisp API uses the `lexdb-` prefix.
- Internal Elisp helpers use `lexdb--` or `lexdb-<adapter>--`.
- Adapter-specific public symbols use `lexdb-ldoce-`, `lexdb-oald-`, or `lexdb-ode-`.
- Do not call adapter-private helpers such as `lexdb-ldoce--...` from other
  files. If logic is genuinely shared, promote it to `lexdb.el` as a
  `lexdb--...` helper with a clear contract.
- Predicate names end in `-p`.
- Unused parameters are prefixed with `_`.
- Python helpers in `scripts/` should use descriptive snake_case names; avoid cryptic abbreviations unless they mirror the source format.

## Elisp Control Flow

- Avoid deep `let` -> `if` -> `let` chains. Favor flat, linear control flow.
- Use `if-let*` and `when-let*` for conditional binding.
- Use `pcase` and `pcase-let` for structured destructuring instead of nested `car`, `cdr`, or `nth`.
- Use `cl-loop` when iteration logic is non-trivial; do not build large accumulators through manual mutation when a clearer loop is available.
- Prefer `cl-loop` over `dolist` plus manual accumulators for non-trivial iteration. `cl-reduce` is acceptable for simple single-operation folds.
- Separate pure data transformation from buffer mutation and side effects whenever practical.
- Interactive commands should stay thin: validate input, call internal logic, then render or message.

## Error Handling

- Errors must surface, not hide. Do not add fallback/default returns that silently swallow failures.
- Catch at the boundary, nowhere else. Only top-level command or adapter boundaries should catch and convert recoverable failures.
- Use `user-error` for user-caused problems such as missing dictionaries, missing DB files, invalid selections, or invalid commands.
- Use `error` for programmer bugs or invariant violations.
- Use `condition-case` only for recoverable failures. Wrap non-essential or optional operations so errors never block primary results.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuciusChen/lexdb](https://github.com/LuciusChen/lexdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
