---
trigger: always_on
description: A general launcher in the terminal. Rust, built on fzf, macOS only.
---

# Working on Prelude

A general launcher in the terminal. Rust, built on fzf, macOS only.
`README.md` describes what it does; this file is what a new session needs to
avoid repeating mistakes already made.

## Build and check

```sh
cargo build --release
cargo test                 # 238 tests
cargo clippy --release     # expected warning-free
./target/release/prelude bench     # p95 gather must stay under 40ms; non-zero when it does not
./target/release/prelude bench --json   # the same distribution, for a gate to record
./target/release/prelude bench --process # a new process per sample: the launch a person meets
./target/release/prelude _dump       # empty-query agent home
./target/release/prelude _dump-root  # searchable root commands
./target/release/prelude _dump-all   # complete catalogue behind scopes
```

`_surface`, `_panel`, `_dump`, `_dump-root`, `_dump-all`, `_footer`, `_focus`, `_preview`,
`_bind`, `_dynamic`, `_copy`, `_runhere`, `_ask`, `_enter`, `_hist`, `_tab`, `_refresh`,
`_refresh-path`, `_refresh-panel-after-update`, `_restart-panel-after-update`, `_copy-skill`,
`_rm-skill`, `_lend-skill`, `_lend-mcp`, `_actions` are internal entry points. They
exist so behaviour can be tested without standing up a terminal — use them
rather than trying to drive fzf.

All launcher entry points use the clipboard surface. The internal doors render
the same labels and action lists without needing a surface environment switch:

```sh
LINE=$(prelude _dump | head -1)
prelude _footer "$LINE"
prelude _actions "$LINE"
```

The agent-facing verbs (`ask --no-wait`, `tell`, `say`, `inbox`, `answer`,
`answer-of`, `fleet --json`) are the same kind of door and are the fastest
way to exercise the bus end to end without a second agent:

```sh
ID=$(prelude ask --no-wait "proceed?")   # what an agent runs
prelude inbox --human                    # what the person sees
prelude answer "$ID" "go ahead"          # the return path
prelude answer-of "$ID"                  # what the agent collects
```

`--human` exists because `whoami()` reads the process tree, and a person
working inside an agent's terminal is correctly identified as that agent —
which would otherwise make their own inbox unreachable from the very window
they are sitting in.

## The launcher panel

`docs/GLOBAL-HOTKEY.md` is the current implementation record. The global
launcher is a **Ghostty quick terminal**: a hidden, dedicated Ghostty instance
configured by `~/.config/prelude/quick-terminal.ghostty`, hosting one long-lived
`prelude _panel` loop. Ghostty registers the chord itself with a `global:`
keybind, so nothing of Prelude's runs when the key is pressed. On macOS that
binding is an Accessibility event tap: installation opens the exact permission
pane and `global status` trusts Ghostty's registration log, never mere process
existence.

**A press reveals; it never creates.** The old design built a terminal on every
press — a new application instance, a window, a login shell — 373ms of
construction, torn down afterwards, including when the answer was a file that
never needed a terminal. Every bug in launch and teardown came from that. There
is now nothing to launch and nothing to strand: the loop outlives every press
and the panel is shown and hidden.

**The bill for that is that the list is older than the press, and `refresh.rs`
is how it is paid.** `once()` starts the launcher *before* the reveal, so
`gather` runs when the previous interaction ended: dismiss at nine, press at
two, and you are reading the morning's machine. The clipping you copied a
minute ago is not in it and will not be until you dismiss and re-open once.
People report this as "it takes a while to show up", which is the wrong shape
— nothing was slow, the snapshot predates what they came for.

A thread therefore re-gathers behind the hidden panel and hands the new list to
fzf through its own `--listen` Unix socket. Three things keep it from being
felt, and each is the part to preserve if this is ever rewritten:

* **Ask fzf before touching it.** The refresh is sent as a `transform`, which
  fzf evaluates with the live `{q}` and `{n}`, so `_tick` can answer with *no
  action* when a query is typed or the cursor has moved. Only fzf knows that;
  guessing would mean a background reload moving somebody's selection, which
  is worse than the staleness it fixes. `may_redraw` treats anything it does
  not recognise as "in use".
* **Do nothing when nothing happened.** A tick is a handful of `stat`s over
  the files behind the list. A gather runs only on a changed mtime, or every
  `FORCE_AFTER` regardless, because a detached slow-source refresh lands in
  its cache with nobody being told.
* **Reuse the layout.** Widths and the title column come from the caller.
  Recomputing them here would put the per-keystroke helper's rows in a
  different column from the static ones — the same trap as everywhere else.

`^K` is a modal: fzf exits, the action panel runs, the list is rebuilt after.
The socket is gone for that whole time, so its absence skips a tick and never
ends the loop. Ending there left every session that had opened `^K` once
silently un-refreshed for the rest of its life.

Everything degrades to nothing: no socket, a failed bind, a POST into the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikewang817/Prelude](https://github.com/mikewang817/Prelude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
