---
trigger: always_on
description: This repository is for an Emacs library that configures Eglot for TypeScript,
---

# AGENTS.md

This repository is for an Emacs library that configures Eglot for TypeScript,
JavaScript, and Astro files.

## Planning

Prefer to write plans in the `plans/` directory.

## Dev loop tools

Here are some strategies to obtain a reliable "dev loop" for validating
integrations.

### Running checks

Run checks against the working tree (no staging required):

```sh
bun run hooks:check
```

This runs the pre-commit hooks against all working tree files with
`--all-files --no-stage-fixed`, so there is no stashing and no auto-staging.
Prefer this for iterating on changes before committing.

### Running tests

Run unit tests with:

```sh
bun run test
```

This executes the default ERT suite found in `test/test.el`.

Run live smoke tests with:

```sh
bun run test:live
```

This runs live `rass` integration tests in parallel (6 concurrent Emacs
processes by default, configurable via `LIVE_TEST_JOBS` env var). It is slower
than `bun run test`, so do not run it by default after every edit.

Run both unit and live tests together with:

```sh
bun run test:all
```

For debugging a single live test failure, use the sequential runner:

```sh
bun run test:live:sequential
```

#### Reading rass event logs

`test/rass-live-client.py --stderr` surfaces rass's event log, but truncates to
4000 chars to keep harness output readable. Event log conventions:

- `[@server_name] <-- method` -- the named server sent a message upstream.
- `<-- method` (no `[@...]` prefix) -- rass forwarded the message to the client
  (Eglot).

For the full (untruncated) stream, write a small debug script that spawns `rass`
directly and drains `proc.stderr` on a background thread. The 4000-char cap in
`rass-live-client.py` is deliberate, not a bug to fix.

Good times to run `bun run test:live`:

- After changing LSP startup or server-contact generation in
  `eglot-typescript-preset.el`
- After changing generated `rass` preset behavior, tool argv handling, or local
  executable resolution
- After changing the live harness itself under `test/`
- Before handing off a change that affects real `rass` integration behavior

Usually `bun run test` plus `bun run check` is enough for docs-only changes,
README edits, or purely static test refactors.

### Packaging checks

Run the MELPA-oriented checks with:

```sh
bun run check:melpa
```

This runs:

- `bun run check`
- `bun run check:doc`
- `bun run check:package-lint`
- `bun run check:melpa:install`

If you only need one part of that checklist, you can run the sub-commands
directly.

Run all MELPA checks plus the full test suite with:

```sh
bun run check:melpa:all
```

This runs `check:melpa` followed by `test:all` (unit + live tests).

The local source of truth for the MELPA recipe is
`eglot-typescript-preset.recipe` at the repository root. The local install check
and Melpazoid workflow both read from that file.

`bun run check:melpa:install` performs a local `package-build` build/install
using a temporary snapshot repository outside the working tree. Keep that temp
root outside this repo because `package-build` may run destructive cleanup in
its own clone/build area.

### Checking for byte-compile warnings

Run the byte-compile check with:

```sh
bun run check
```

This byte-compiles `eglot-typescript-preset.el` and fails if there are any
warnings. Fix all warnings before committing.

### Introspecting Elisp functions/variables from CLI

To check a function's arguments or documentation without starting interactive
Emacs:

- Get argument list (using `eglot` as an example):
  - `emacs -Q --batch --eval "(require 'eglot)" --eval "(princ (help-function-arglist 'eglot))"`
- Get function documentation (using `eglot` as an example):
  - `emacs -Q --batch --eval "(require 'eglot)" --eval "(princ (documentation 'eglot))"`
- Get variable documentation (using `eglot-server-programs` as an example):
  - `emacs -Q --batch --eval "(require 'eglot)" --eval "(princ (documentation-property 'eglot-server-programs 'variable-documentation))"`

### Quick Elisp sanity check

- Check parentheses balance after edits:
  - `emacs -Q --batch --eval '(progn (with-temp-buffer (insert-file-contents "eglot-typescript-preset.el") (check-parens)))'`

### One-off batch harnesses

When iterating on a small part of the config (a single function, hook, or
integration), prefer writing a tiny one-off `.el` file under `tmp/` and running
it via `emacs -Q --batch`. This keeps the feedback loop fast without requiring a
full interactive Emacs session.

- Example workflow:
  - Create `tmp/<topic>-test.el` that loads just what you need (either by
    copying the relevant forms or by selectively `load-file`-ing a small file).
  - Run it:
    - `emacs -Q --batch -l tmp/<topic>-test.el`
  - Print output with `princ` and exit non-zero with `(kill-emacs 1)` on
    failure.

## Gotchas

### JSON serialization

When working with JSON serialization issues (especially with LSP
configurations):

- **Test individual components**: Isolate problematic parts by testing JSON
  serialization of individual plists and values to identify the exact issue.

- **Use plists for JSON objects**: `json-serialize` can't handle lists
  containing other lists, but can handle plists:

  ```elisp
  ;; This fails:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mwolson/eglot-typescript-preset](https://github.com/mwolson/eglot-typescript-preset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
