---
trigger: always_on
description: Notes for agents (and humans) changing this codebase. This file covers the
---

# Working on local-operator

Notes for agents (and humans) changing this codebase. This file covers the
things that are easy to get wrong here and expensive to discover later. For
what the rewrite set out to do see `docs/REWRITE.md`; for the evidence behind
each round see `docs/VERIFICATION.md`.

## Environment

```sh
cd ~/local-operator
.venv/bin/python -m pytest tests/unit -q          # ~2700 tests, ~3.5 min
```

TUI tests need a colour-capable terminal, so run them with the environment the
suite expects:

```sh
env -u NO_COLOR TERM=xterm-256color .venv/bin/python -m pytest tests/unit/tui -q
```

`tests/e2e` is a separate stage that drives the **assembled** application —
boot, a real turn through a real tool, and `/resume` — and it is **deselected
from the default run** (`-m "not e2e"` in `addopts`). Run it explicitly:

```sh
env -u NO_COLOR TERM=xterm-256color .venv/bin/python -m pytest tests/e2e -m e2e -n0 -q
```

It exists because the whole unit suite was green while the TUI was completely
frozen (#401: a blocking `flock` in the MCP OAuth refresh lock deadlocked the
event loop on `/resume`). Two things about it are load-bearing rather than
stylistic, and both are documented at length in `tests/e2e/watchdog.py`:

- **Its failure mode is a hang, not an assertion.** The deadlock parks two
  threads inside syscalls, so `asyncio.wait_for`, thread watchdogs and
  signal-based timeouts all fail to fire — verified, not assumed. Only
  `faulthandler.dump_traceback_later(exit=True)` survives it, because it runs
  in a C thread and needs no GIL. A tripped watchdog kills the process and
  writes every thread's stack to a file; `python -m tests.e2e.watchdog` prints
  it back.
- **It is why the stage is deselected and runs `-n0`.** Under xdist a fired
  watchdog would kill a worker carrying unrelated tests and report them as an
  infrastructure error rather than as the freeze they are.

It is fully headless (Textual's `run_test()` pilot, no window, no display, no
TTY) and uses no API key, so its CI job carries **no fork gate** — unlike
`cli-sanity`/`server-sanity`, whose live-LLM secrets force one. That is
deliberate: the resume-liveness assertion is the regression guard, so it has to
run on every PR including forks.

**It runs on a `[ubuntu-latest, macos-latest]` matrix, and the macOS leg is the
one that makes it a regression guard.** The deadlock is a macOS/BSD property —
`close()` blocks there behind a sibling `flock()`, and on Linux it returns in
microseconds. Measured, not assumed: the same probe reports `close_blocked=True`
on darwin and `False` on linux, and the resume test run against the pre-fix tree
(`80df237b^`) in `python:3.12-slim` reports `1 passed`. Linux still runs the
stage because boot, the write-turn artifacts and the transcript replay are
platform-neutral, but **do not drop the macOS leg** — without it this stage goes
green against the exact commit it exists to catch.

Gates, all of which must be clean before a PR. **Run them over the whole tree,
exactly as CI does** — these are the commands from `.github/workflows/ci.yml`:

```sh
.venv/bin/python -m flake8 .
uvx --from black==26.1.0 black --check .
uvx isort==5.13.2 --check .
.venv/bin/python -m pyright --pythonpath .venv/bin/python .
```

Narrowing the last two to `local_operator tests`, or passing `--profile black`
instead of letting isort read the repo's own config, checks something CI does
not: both combinations pass on a file that CI then rejects. An unsorted
function-local import reached CI exactly that way.

Editing `exclude` under `[tool.pyright]` in `pyproject.toml`? It **replaces**
pyright's built-in defaults rather than extending them, so always restate
`"**/node_modules"`, `"**/__pycache__"` and `"**/.*"` alongside whatever you
are adding. Dropping `**/.*` makes pyright follow the `.venv` symlink every
worktree has and type-check all of site-packages — 29466 files and a 30-minute
run instead of 566 files and about 15 seconds. CI never creates a `.venv`, so
it stays green while every local run of the gate becomes unusable.

The venv is uv-managed and has the package installed **editable**, so source
edits are live. After a pull that changes dependencies:

```sh
uv pip install -e ".[all,dev]" --python .venv/bin/python
```

## Releasing the stable `lop` runtime

Development and the global launcher deliberately use different installations:

- `uv run local-operator` and `.venv/bin/local-operator` execute the current
  checkout. Use them while developing and validating source changes.
- `lop` executes the non-editable uv tool installation under
  `~/.local/share/uv/tools/local-operator`. It must remain independent of the
  checkout so branch switches and uncommitted work cannot break the global TUI.

After a change is tested and committed to `main`, make it live with:

```sh
lop-update
```

`lop-update` archives the committed `main` ref, builds and installs that
snapshot, and records the exact source revision in
`~/.local/share/uv/tools/local-operator/.lop-source`. It never packages the
currently checked-out branch or uncommitted files. A specific committed ref can
be installed deliberately with `lop-update <git-ref>`.

Every agent asked to "update local-operator" or make a change available through

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damianvtran/local-operator](https://github.com/damianvtran/local-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
