---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

legmacs is a terminal text editor written in and scriptable with
[let-go](https://github.com/nooga/let-go) (a Clojure-dialect Lisp with a Go
VM). It's Emacs-flavored: chord-based keys (`C-x C-s`), a minibuffer, a
kill-ring, major modes. There is no separate plugin language — the built-in
commands and keymap are defined with the exact same API a user's own config
uses.

The sibling checkout `../let-go` is the language/runtime itself (not part of
this repo). If a builtin's behavior is unclear, its Go source is more
authoritative than guessing: `pkg/rt/lang.go` (builtins), `pkg/rt/core/*.lg`
(stdlib written in let-go), `pkg/compiler/eval.go` (`eval`/`read-string`/
`load-string`/`read-all-string`), `pkg/rt/term.go` (the `term` namespace —
raw mode, ANSI, key reading).

## Commands

Run it (from this directory, so namespace resolution works — see Gotchas):

```sh
lg main.lg [file...]
```

Or via the installed wrapper, which works from any cwd and picks up
`~/.config/legmacs/legmacs_init.lg`:

```sh
bin/legmacs [file...]
```

Run the full test suite:

```sh
lg test/run.lg
```

Run a single test namespace (faster iteration than the full suite):

```sh
lg -e "(require '[test :refer [run-tests]] '[test.region-test]) (run-tests)"
```

There's no separate build/lint step — `lg` interprets `.lg` source directly.
`test/run.lg` is a hand-maintained list of every test namespace; adding a
new `test/*_test.lg` file means adding its namespace to that `:require`
list or it silently won't run.

## Architecture

**The pipeline is a pure state machine.** Everything above `main.lg` is
`(fn [state] -> state)` or `(fn [state] -> string)` — no I/O, no atoms for
the document itself. `main.lg`'s `loop`/`recur` is the only place that
touches the terminal:

```
term/read-key --(legmacs.keys)--> chord string
  --(legmacs.dispatch, consulting legmacs.keymap + legmacs.modes)--> command
    --(legmacs.commands, built on legmacs.buffer)--> new state
      --(legmacs.render)--> one ANSI string --(term/write)--> screen
```

This is what makes the whole editor unit-testable without a TTY (see
`test/`) — buffer edits, key parsing, dispatch, even rendering are just pure
functions checked against expected data.

**Scripting = the same registries the built-ins use.** `legmacs.keymap`
holds two atoms: a command registry (name → fn) and the keymap tree
(chord-path → command). `defcommand` defines a normal function *and*
registers it; `bind-key!` attaches a chord sequence to a registered command.
`legmacs/bindings.lg` (the defaults) and a user's `legmacs-init.lg` call the
exact same functions — there's no privileged built-in path.

**Modes shadow the global keymap key-by-key, not sequence-by-sequence.**
`legmacs.modes` lets a mode add a leaf under an existing global prefix
(e.g. let-go-mode's `C-x C-e`) without hiding the rest of that prefix's
global bindings — `legmacs.dispatch` walks *all* applicable keymaps in
parallel at each chord, not one keymap chosen up front. A mode can also
supply `:highlighter` (pure per-line syntax coloring, called fresh every
frame for visible lines only) and `:after-command` (runs after every
dispatch while that mode is active — used for paren-matching). `legmacs/
lisp_syntax.lg` is a single-pass bracket/string/comment scanner shared by
paren-matching, auto-indent, expand-region, and auto-pairing in
`legmacs/modes/letgo.lg`; `legmacs/modes/markdown.lg` is a second,
independent mode (highlighting only) built the same way; and `legmacs/
modes/prog.lg` generalizes that shape into one spec-driven per-line scanner
(comment markers, string delimiters, keyword/type/constant word sets)
behind major modes for Go/JS/Python/C/shell/Rust/JSON/YAML/TOML/Lua/Ruby/
SQL/Dockerfile/CSS/HTML/Zig/Java/Kotlin/Swift/C#/PHP/Makefile — its
`register-prog-mode!` is both how the built-ins
register and the user-facing one-call way to add a language. `:block-comment`
is checked before `:line-comments` in both this scanner and
`legmacs/prog_syntax.lg` below, since a block-open marker can be a
strict-prefix superstring of the line-comment marker (Lua: `--[[` vs `--`).

**Paren-matching, auto-pairing, and auto-indent are reusable minor modes,
not let-go-only.** `legmacs/prog_syntax.lg` is a sibling of
`legmacs/lisp_syntax.lg` — the same full-buffer bracket/string/comment scan,
but driven by a language `:spec` map (the exact one `register-prog-mode!`
already builds a highlighter from: `:line-comments`, `:block-comment`,
`:string-delims`, plus purely-structural `:brackets`/`:indent-width`
knobs) instead of Lisp's fixed rules. `legmacs/modes/structural.lg` builds
three ordinary minor modes on top of it — `:paren-match`, `:electric-pair`,
`:auto-indent` — that look up `(modes/mode-syntax-spec (:mode state))` at
call time and degrade to plain behavior (self-insert, plain newline, clear
the match) when the active buffer's major mode has none. `register-prog-mode!`
stores its spec as the mode's `:syntax-spec`, and `legmacs.modes/switch-to-mode`
auto-enables all three whenever the mode being switched to has one — so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nooga/legmacs](https://github.com/nooga/legmacs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
