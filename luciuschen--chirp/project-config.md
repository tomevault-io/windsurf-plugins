---
trigger: always_on
description: This file applies to the entire repository. Keep it self-contained: agents should not need to read another repository to learn Chirp's rules.
---

# Chirp Agent Guide

This file applies to the entire repository. Keep it self-contained: agents should not need to read another repository to learn Chirp's rules.

## Product and Architecture Boundaries

- Chirp is an Emacs UI for X/Twitter. `twitter-cli` owns authentication, network access, API compatibility, and wire-format details; do not add direct X API calls to Chirp.
- `chirp.el` is the public entry point. External users load `(require 'chirp)`.
- `chirp-backend.el` owns `twitter-cli` discovery, process invocation, retries, and JSON envelope handling.
- `chirp-core.el` owns shared state, history, buffer lifecycle, and cross-view navigation.
- `chirp-render.el` renders normalized data. `chirp-media.el` owns cache paths, thumbnail extraction, prefetching, and large-media display.
- View modules orchestrate fetching and rendering; they must not duplicate backend, normalization, or media behavior. `chirp-actions.el` owns compose and write actions, which share one backend request path.

## Change Discipline

- Fix the layer that owns the problem. Name the failing boundary before changing behavior; do not compensate with timing changes, duplicate lookups, or silent fallbacks elsewhere.
- Prefer the smallest coherent implementation. Do not add layers, files, state objects, or compatibility paths for hypothetical future needs.
- Treat helper stacks as debt. Inline trivial one-use wrappers and collapse pass-through ladders; retain a helper only when it owns a complete calculation or workflow and makes its caller materially clearer.
- Aim to keep functions around 30 lines, but do not manufacture tiny wrappers solely to meet a line count. First simplify state, data flow, and control flow.
- Delete unused code and obsolete tests outright. Do not leave deprecated aliases, commented-out paths, or tests that only keep dead helpers alive.
- Refactors must reduce duplication, centralize a real invariant, simplify callers, or improve robustness. Renaming or moving code alone is not enough.
- Read the surrounding implementation, tests, documentation, and integration boundary before changing a user-visible workflow.

## Emacs Lisp Conventions

- The Emacs baseline is 29.1. Verify newer APIs before use and do not raise the baseline without updating package metadata, documentation, and the changelog.
- Every `.el` file uses lexical binding, has the correct package prefix, provides its feature, and ends with the standard footer.
- Public API uses `chirp-`; private implementation uses `chirp--` or the owning module's double-dash prefix. Never call another package's private symbols.
- Use lowercase hyphenated Lisp names. Single-word predicates end in `p`; multi-word predicates end in `-p`. Prefix intentionally unused lexical variables and arguments with `_`.
- Prefer face names without a redundant `-face` suffix for new APIs, but preserve Chirp's established public face naming instead of renaming customization symbols only for style.
- Let Emacs indentation be authoritative. Use spaces rather than hard tabs, keep trailing parentheses together, separate unrelated top-level forms with a blank line, and keep lines near 80 characters when that improves readability without awkwardly splitting clear strings, URLs, or forms.
- Use `when` for one positive branch, `unless` for one negated branch, `not` for boolean negation, and `null` specifically for the empty list. Do not add a redundant `progn`. Prefer chained comparisons and `1+`/`1-` idioms.
- Use `defvar-local` for per-buffer state, `defcustom` with a precise `:type` and `:group` for user options, and plain `defvar` only for shared process-wide state.
- Loading files must not change the user's active editing behavior. User-facing commands and modes activate behavior explicitly.
- Prefer flat control flow with `if-let*`, `when-let*`, `pcase`, and `pcase-let`. Prefer stock Emacs protocols and primitives over custom frameworks.
- Prefer `cl-loop` over `dolist` plus manual accumulation for non-trivial iteration. Do not use `mapcar` when its result is discarded; use `dolist` for multi-form side effects and `seq-do` for applying one existing function.
- Prefer idiomatic primitives over reconstructed equivalents, such as `vconcat` for vectors and direct predicate results instead of `(not (null ...))`.
- Prefer `let*`, `pcase-let`, alists/plists, small helpers, or table-driven mappings for short-lived context. Avoid more than three or four positional parameters; use `cl-defun` keyword arguments or one documented plist/alist when order becomes hard to remember. Reserve structs or object layers for stable state crossing module or lifecycle boundaries.
- Use `#'function-name` for executable function values. Use lambdas only for genuinely local behavior; hooks, keymaps, advice, customizable callbacks, and other long-lived registrations normally use named functions. Do not wrap a function in a lambda that only forwards the same arguments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuciusChen/chirp](https://github.com/LuciusChen/chirp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
