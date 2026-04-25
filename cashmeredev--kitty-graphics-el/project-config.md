---
trigger: always_on
description: Pure Emacs Lisp package (`kitty-graphics.el`) for displaying images in
---

# AGENTS.md -- kitty-graphics

## Project Overview

Pure Emacs Lisp package (`kitty-graphics.el`) for displaying images in
terminal Emacs (`emacs -nw`) using either the Kitty graphics protocol or
Sixel, with automatic protocol detection. Also renders org-mode headings
at scaled font sizes using the Kitty text sizing protocol (OSC 66).
Integrates with org-mode, image-mode, shr/eww/mu4e/gnus, dired, and dirvish.

Language: **Emacs Lisp** (`lexical-binding: t`). Single source file (~1876 lines).
Current version: **0.3.0** (see Versioning section).

## Build / Validate

### Byte-compile (primary lint check -- run after every change)

```sh
emacs -Q -batch -f batch-byte-compile kitty-graphics.el
```

**This is the closest thing to CI** -- always run it before considering
a change complete.

### Load-test (verify file evaluates without error)

```sh
emacs -Q -batch -l kitty-graphics.el -f kill-emacs
```

## Testing

No automated test framework. All testing is manual/interactive and
requires a supported terminal (Kitty, WezTerm, Ghostty for Kitty backend;
foot, Konsole, xterm, mlterm, mintty for Sixel backend).
The `tests/` directory contains `test-kitty-gfx.org`, `test-image.png`,
and `test-document.pdf` for manual testing -- use these rather than
creating new test files.

**Important**: Always set `TERM=xterm-256color` when launching terminal
Emacs -- the `xterm-kitty` terminfo is often missing.

### Test with org-mode

```sh
TERM=xterm-256color emacs -nw -Q -l kitty-graphics.el \
  --eval "(kitty-graphics-mode 1)" tests/test-kitty-gfx.org
# Then: C-c C-x C-v (org-toggle-inline-images)
```

### Test text sizing (org heading scales)

```sh
TERM=xterm-256color emacs -nw -Q -l kitty-graphics.el \
  --eval "(setq kitty-gfx-heading-sizes-auto t)" \
  --eval "(kitty-graphics-mode 1)" tests/test-kitty-gfx.org
# Headings should render at scaled sizes automatically.
# S-TAB to unfold all, then scroll to verify fold/unfold/scroll.
```

### Test image-mode

```sh
TERM=xterm-256color emacs -nw -Q -l kitty-graphics.el \
  --eval "(kitty-graphics-mode 1)" tests/test-image.png
```

### Debug logging

Set `kitty-gfx-debug` to `t` to log debug info to `/tmp/kitty-gfx.log`.

### Dev environment

The repo uses a Nix flake (`.envrc` → `use flake`). If you have direnv
and Nix, dependencies are provided automatically. This is optional --
only `emacs` and optionally `imagemagick` are needed.

### Agent workflow

When you need the user to manually test a feature in a Kitty-compatible
terminal, or have any question that requires user action, notify them:

```sh
notify-send "kitty-graphics" "Please test: <description of what to test>"
```

## Code Style

### File structure (mandatory order)

```
;;; file.el --- Description -*- lexical-binding: t; -*-
;;; Commentary:
;;; Code:
(require ...)           ; dependencies
(declare-function ...)  ; forward declarations for optional deps
(defgroup ...)          ; customization group
(defcustom ...)         ; user options
(defconst ...)          ; constants
(defvar / defvar-local) ; internal state
;; ... implementation sections with ;;;; headers ...
(provide 'feature)
;;; file.el ends here
```

### Formatting

- **Spaces only**, no tabs (`indent-tabs-mode: nil`; enforced by `Local Variables` block)
- `lexical-binding: t` required in the first line
- `;;;;` section headers to separate logical sections

### Naming conventions

| Category              | Pattern                        | Example                          |
|-----------------------|--------------------------------|----------------------------------|
| Public API            | `kitty-gfx-` prefix           | `kitty-gfx-display-image`       |
| Internal functions    | `kitty-gfx--` (double dash)   | `kitty-gfx--transmit-image`     |
| Buffer-local vars     | `defvar-local` + double dash  | `kitty-gfx--overlays`           |
| Customization vars    | `defcustom` + single dash     | `kitty-gfx-max-width`           |
| Global internal vars  | `defvar` + double dash        | `kitty-gfx--next-id`            |
| Advice functions      | `kitty-gfx--MODE-VERB-advice` | `kitty-gfx--org-display-advice` |
| Minor mode            | `kitty-graphics-mode`          |                                  |

All identifiers use `kebab-case`.

### Requires / Dependencies

- Only `cl-lib` is `require`d directly
- All other dependencies (org, image-mode, shr, dired, dirvish) use
  `declare-function` for forward declarations and `with-eval-after-load`
  for deferred integration -- **never `require` optional packages at
  top level**

### Error handling

- `user-error` for user-facing errors (e.g., unsupported terminal)
- `condition-case` around I/O (terminal queries, files, image processing)
- `ignore-errors` around `send-string-to-terminal` -- terminal writes
  must never crash Emacs
- `when`/`unless` guards rather than explicit error branches
- `unwind-protect` for cleanup (see `kitty-gfx--refresh`)

### Advice convention

All advice uses `:around` and guards on both mode and terminal:

```elisp
(defun kitty-gfx--FOO-advice (orig-fn &rest args)
  (if (and kitty-graphics-mode (not (display-graphic-p)))
      ... ;; terminal path
    (apply orig-fn args)))
```

New mode integrations **must** follow this pattern. Each mode needs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cashmeredev/kitty-graphics.el](https://github.com/cashmeredev/kitty-graphics.el) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
