---
trigger: always_on
description: This file is the local source of truth for AI-assisted changes in this
---

# AGENTS.md

This file is the local source of truth for AI-assisted changes in this
repository. It is adapted from `~/repos/coding-guidelines/general.md` and
`~/repos/coding-guidelines/elisp.md`, plus project-specific ytm-radio rules.

## Project Shape

- Keep the project small until real complexity appears. Prefer one clear file
  with well-named sections over several tiny files with unclear boundaries.
- Split modules only around stable responsibilities such as state, external
  process management, source fetching, helper protocol handling, or UI display.
- Do not add abstraction layers for hypothetical providers. Add a layer only
  when it removes current duplication or gives a real owner to a workflow.
- Do not create vague `utils`, `common`, or pass-through wrapper modules.
- Keep public commands thin: collect interactive input, validate it, call
  internal functions, and show feedback.
- Prefer stock Emacs primitives: `completing-read`, `special-mode`, text
  properties, `start-process`, `make-network-process`, standard timers, and
  standard hooks.

## Diagnosis Discipline

- Find the root cause before changing behavior. Be able to name the failing
  layer before patching timing, caching, rendering, or control flow.
- Do not present a plausible explanation as root cause. Mark it as a
  hypothesis until code inspection, a reproduction, or a failing test confirms
  the actual failing path.
- If one fix fails, narrow the hypothesis and gather more evidence. Do not
  stack another speculative patch on top.
- After two failed fixes on the same issue, stop patching and switch to
  diagnosis only until the failing path is confirmed.
- Fix the layer that owns the problem instead of compensating elsewhere.
- Keep experiments narrow. Prove a new direction with the smallest useful
  slice before expanding scope.
- For dispatcher bugs, test the real dispatch path: keymaps, buttons,
  commands, hooks, async callbacks, and public entry points. Helper-level tests
  are not enough when the bug is in routing.
- For user-visible bug fixes, prefer red before green: reproduce the failure in
  a test or a minimal live check, confirm it fails, then change behavior.
- Do not leave heuristic shortcuts, silent partial implementations, duplicated
  logic, or dead code introduced during diagnosis.

## Emacs Lisp Rules

- Every `.el` file uses lexical binding.
- Loading package files must not alter active editing behavior. Activation
  happens through explicit commands or user-enabled modes.
- Use the `ytm-radio-` prefix for public API and `ytm-radio--` for private
  helpers and private modes.
- Never call another package's private double-dash symbols.
- Public commands and user-facing modes need `;;;###autoload`.
- Do not autoload internal helpers, variables, or private modes.
- Public `defun`, `defmacro`, `defcustom`, and `defvar` forms must have
  docstrings.
- Docstring first lines must be complete sentences ending in a period.
- Argument names mentioned in docstrings should be uppercased.
- Use precise `defcustom :type` declarations and always set `:group`.
- Use `defvar-local` and `setq-local` for per-buffer state. Major modes must
  make their state buffer-local.
- Read-only UI buffers derive from `special-mode`.
- Use text properties for data-bearing annotations; use overlays only for
  ephemeral visuals.
- Build render buffers from structured state, not by reparsing visible text.
- Prefer `when-let*`, `if-let*`, `pcase`, and `pcase-let` for structured
  conditional binding and destructuring.
- Use `user-error` for user-caused problems such as missing external programs,
  invalid configuration, or empty catalogs.
- Use `error` for programmer bugs. Catch errors only at external process,
  optional display, or top-level helper protocol boundaries where recovery is
  meaningful.
- Require runtime dependencies explicitly, for example `(require 'cl-lib)`.
  Do not rely on transitive loading.
- Avoid `eval-when-compile` for dependencies needed at runtime.
- Before using a newer Emacs API, verify when it was introduced and do not
  exceed the declared Emacs baseline without updating package metadata and docs.

## MELPA / Package Rules

- Main package first line must be:
  `;;; ytm-radio.el --- Short description -*- lexical-binding: t; -*-`
- The package description must not contain "for Emacs" or the package name.
  Keep it under 60 characters.
- The main package file must include `;; Author:`, `;; URL:`, `;; Version:`,
  and `;; Package-Requires:`.
- `Package-Requires` must list all direct dependencies with minimum versions,
  including the declared Emacs baseline.
- Package metadata belongs in the main package file only. Split implementation
  files must not duplicate `Package-Requires`.
- Split implementation files still need formal license metadata, preferably
  `;; SPDX-License-Identifier:`.
- Keep required MELPA checklist attribution such as `;; Assisted-by: ...` in
  the main package file when tooling materially assisted the package.
- Every distributable `.el` file ends with `(provide 'feature)` and
  `;;; file.el ends here`.
- Run byte-compilation with zero warnings.
- Run `checkdoc` with zero warnings on distributable Elisp files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuciusChen/ytm-radio](https://github.com/LuciusChen/ytm-radio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
