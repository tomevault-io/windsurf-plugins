---
trigger: always_on
description: Guidance for agentic coding agents operating in this repository.
---

# AGENTS.md — SEM Assistant Elisp Daemon

Guidance for agentic coding agents operating in this repository.

---

## Project Overview

A self-hosted Emacs daemon (Docker) that autonomously processes Org inbox notes
and RSS feeds via LLM (OpenRouter/gptel). All logic is written in Emacs Lisp.
Source lives in `app/elisp/`. Tests live in `app/elisp/tests/`.

---

## Commands

### Run all tests

```sh
eask test ert app/elisp/tests/sem-test-runner.el
```

### Run a single test file

```sh
eask emacs --batch \
  --load app/elisp/tests/sem-mock.el \
  --load app/elisp/tests/sem-core-test.el \
  --eval "(ert-run-tests-batch-and-exit)"
```

Replace `sem-core-test.el` with the file under test. Always load
`sem-mock.el` first so behavioral mocks are available.

### Run a single named test

```sh
eask emacs --batch \
  --load app/elisp/tests/sem-mock.el \
  --load app/elisp/tests/sem-core-test.el \
  --eval "(ert-run-tests-batch-and-exit 'sem-core-test-cursor-roundtrip)"
```

### Check unmatched parentheses / brackets

```sh
sh dev/elisplint.sh app/elisp/sem-core.el
```

Pass multiple files at once:

```sh
sh dev/elisplint.sh app/elisp/sem-core.el app/elisp/sem-router.el
```

The linter uses `scan-sexps` via Emacs batch mode and reports the line of the
first unmatched delimiter. Run this before committing any elisp change.

### Start the daemon (Docker)

```sh
docker-compose up -d
docker-compose logs -f emacs
```

---

## Repository Layout

```
Eask                    Dependency manifest for Emacs packages
app/elisp/              Source modules (load order matters — see init.el)
  init.el               Daemon startup sequence
  sem-core.el           Logging, cursor tracking, retry, inbox purge
  sem-security.el       Sensitive-block masking, URL sanitization
  sem-llm.el            gptel wrapper (all LLM calls go here)
  sem-prompts.el        Prompt templates and cheat-sheets
  sem-router.el         Inbox parsing and routing
  sem-url-capture.el    URL → org-roam pipeline
  sem-rss.el            RSS/arXiv digest generation
  sem-git-sync.el       Automated org-roam → GitHub sync
  tests/                ERT test suite
    sem-mock.el         Shared mock helpers (load first)
    sem-test-runner.el  Full suite runner
dev/
  elisplint.sh          Parenthesis linter
```

---

## Elisp Code Style

### File Header

Every source file must start with:

```elisp
;;; filename.el --- Short description -*- lexical-binding: t; -*-
;; SPDX-License-Identifier: GPL-3.0-or-later

;;; Commentary:
;; One or two paragraphs describing the module.

;;; Code:
```

And end with:

```elisp
(provide 'module-name)
;;; filename.el ends here
```

### Lexical Binding

All files use `lexical-binding: t` in the file-local variables line. Never omit
this. Dynamic binding bugs are subtle and hard to trace.

### Requires / Imports

- `require` statements appear at the top of `;;; Code:`, one per line.
- Modules required only inside specific functions use `(require 'foo)` locally
  (lazy loading pattern used in `sem-router.el` and `sem-core.el`).
- Do not use `load` or `load-file` in source modules — only in test runners.

### Naming Conventions

| Pattern | Example | Usage |
|---|---|---|
| `module-name-public-fn` | `sem-core-log` | Public API, callable from other modules |
| `module-name--private-fn` | `sem-core--read-cursor` | Private, internal only |
| `module-name-constant` | `sem-core-log-file` | `defconst`, module-level constant |
| `module-name--var` | `sem-router--tasks-write-lock` | `defvar`, mutable state |

- Use `defconst` for values that never change at runtime.
- Use `defvar` for mutable module-level state; always provide a docstring.
- Function names are lowercase, hyphen-separated. No camelCase.

### Formatting

- 2-space indentation for body forms inside `let`, `progn`, `when`, `cond`, etc.
- Closing parentheses are never on their own line (Lisp style, not Allman).
- `let*` is preferred over nested `let` when bindings are sequential.
- Keep lines under ~100 characters. Long strings (prompts) may exceed this.
- Leave one blank line between top-level forms. Use section comments (`;;; Section`).

### Docstrings

- All `defun`, `defvar`, `defconst` must have docstrings.
- First line is a complete sentence ending with a period.
- Document all parameters by name in ALL-CAPS (Emacs convention).
- For callbacks, document the callback signature inline (see `sem-router.el`).

### Types and Data

- Plists are used for headline data: `(list :title "..." :tags '("task") :hash "...")`.
- Alists are used for cursor/retry tracking: `'(("hash" . t))`.
- Prefer `plist-get` / `plist-put` over positional list access.
- Use `cl-lib` functions (`cl-block`, `cl-return-from`, `cl-remove-if-not`,
  `cl-letf`, etc.) — `cl-lib` is always required in source files.

---

## Error Handling

### The golden rule: never let the daemon crash

- Every public entry point and every cron-callable function must be wrapped in
  `(condition-case err ... (error ...))`.
- Logging functions themselves are wrapped in `(ignore-errors ...)` or
  `(condition-case _err ... (t nil))` — they must never propagate errors.
- Catch errors at the outermost layer; log with `sem-core-log-error`, then
  continue. Do not re-signal.

### Error logging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SemyonSinchenko/sem-assistant-el](https://github.com/SemyonSinchenko/sem-assistant-el) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
