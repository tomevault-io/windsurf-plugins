---
trigger: always_on
description: A short guide for AI agents (and the humans who wire them up) that want to
---

# tgrep for coding agents

A short guide for AI agents (and the humans who wire them up) that want to
use `tgrep` as a fast search tool inside a repository. It complements the
[README](README.md), which documents every flag; this file covers the few
things an agent has to get right.

## The mental model

tgrep is ripgrep with a pre-built trigram index and an optional server.

```
tgrep index .        # once: build the index into ./.tgrep
tgrep serve .        # once per session: keep the index warm and watch for changes
tgrep "pattern" .    # every search: finds the server, answers in milliseconds
```

A search resolves in this order:

1. **Server** running for this tree: query it over TCP. Fastest. A file
   watcher keeps the index close to the filesystem, though a silently missed
   notification is repaired only by periodic reconciliation (scheduled hourly
   and deferrable for up to four hours while queried); `--no-watch` disables it.
2. **On-disk index** but no server: read `.tgrep/` directly. Fast, but only as
   fresh as the last successful index publication. Legacy indexes without
   hidden-file coverage metadata and incomplete builds fall back to scanning.
   Rebuild with `tgrep index .` or resume `tgrep serve .` to enable indexed queries.
3. **No index**: scan every file, like grep. Correct but slow on large trees.
   tgrep prints a warning on stderr when this happens.

An agent never has to choose between these; the command is the same. Results
can differ, though. An on-disk index omits changes since its last build. A
server started with no index, a partial index, or legacy hidden-file coverage
lets clients fall back to scanning until it establishes the full corpus.
`tgrep status .` shows `Hidden coverage: complete` once indexed queries are
available, and `Indexing: complete` once the initial build is done. Do not read either as a
freshness signal: a server that starts on an existing index reconciles it
against the filesystem in the background while already reporting complete,
and it never covers watcher events missed later. When a search must reflect current file
contents, pass `--no-index`. It reads every eligible file from disk instead of
consulting the index, still applying the normal ignore, hidden-file, binary
and size rules. It is slow on large trees, so use it deliberately.

## Setup

```bash
brew install tgrep                      # macOS, Linux
cargo install --path tgrep-cli --locked # from a checkout
```

Then, from the repository root, once:

```bash
tgrep serve . &
```

`serve` builds the index if none exists and answers queries while it builds.
It writes `.tgrep/serve.json` (PID and port) so clients can find it. Do not
commit `.tgrep/`; add it to `.gitignore`.

Check that a server is up:

```bash
tgrep status .
```

If your agent framework cannot keep a background process alive, skip `serve`
and run `tgrep index .` instead. Searches then use the on-disk index. That
index is not updated by searches or edits, so re-run `tgrep index .` after any
change a later search has to see, including your own edits.

## Searching

The command line follows ripgrep. The common `rg` flags are supported with the
same names; an unsupported flag is rejected with an error rather than ignored.
The full list is in the [README](README.md#cli-flags). Three accepted flags
only take effect on a full scan: `-L`/`--follow`, `--one-file-system` and
`--ignore-file`. An indexed search ignores them silently, so pair them with
`--no-index`.

```bash
tgrep -- "fn parse_config" .                 # regex, default
tgrep -F -- "Vec<Option<T>>" .               # literal string
tgrep -w -t rust -- handle .                 # whole word, Rust files only
tgrep -g "src/**" -C 2 -- "TODO|FIXME" .     # glob scope, 2 lines of context
tgrep -l -- "impl .* for Server" .           # file names only
tgrep -c -- deprecated .                     # count per file
tgrep --files -t py .                        # list searchable Python files
```

Rules of thumb for agents:

- **Put `--` before the pattern** and pass the search root explicitly. Shell
  quotes do not stop the parser from reading a bare `index`, `serve`,
  `search`, `status`, `count-files` or `help` as a subcommand;
  `tgrep -- serve .` searches for the word. Everything after `--` is read as
  the pattern and paths, so all flags must come before it.
- **Prefer `-F`** when the query is a symbol or a string the user typed. It
  avoids regex-escaping mistakes.
- **Narrow with `-t` or `-g`** before adding `-m`. Both keep compatible indexes
  in use; globs filter the indexed corpus rather than reinclude ignored files.
  `-m` only trims output.
- **Use `-l` first** on a broad query, then search the specific files. This
  keeps output small.
- **Use `-C 2` or `-C 3`** when you need to read the surrounding code.
- **Use `-q`** when you only need a yes/no answer; read the exit code.

## Machine-readable output

`--json` emits one JSON object per line, in ripgrep's format. Record types
are `begin`, `match`, `context`, `end`, and `summary`.

Real output, run from a checkout of this repository:

```bash
tgrep --json -F -- "fn main" tgrep-cli/build.rs
```

```json
{"data":{"path":{"text":"tgrep-cli/build.rs"}},"type":"begin"}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/tgrep](https://github.com/microsoft/tgrep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
