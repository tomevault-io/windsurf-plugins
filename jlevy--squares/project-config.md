---
trigger: always_on
description: Use when the user asks to improve, audit, score, or compare practical documents.
---

# Project Instructions for AI Agents

This file provides instructions and context for AI coding agents working on this
project.

<!-- BEGIN TBD INTEGRATION format=f08 surface=agents-md -->
## tbd

This repository uses **tbd** for git-native issue tracking (beads), spec-driven
planning, and on-demand engineering guidelines.
As the agent, you operate tbd on the user’s behalf: translate their requests into tbd
actions rather than telling them to run commands.

- Run `tbd prime` to load current project state and the full tbd workflow.
- Run `tbd skill` for the complete reusable tbd skill instructions.
- Run `tbd shortcut --list` and `tbd guidelines --list` for on-demand resources.
- Track all work as beads: `tbd create`, `tbd ready`, `tbd close`, and `tbd sync`.

<!-- END TBD INTEGRATION -->

## Operating Rules

Generated from [`operating-rules.md`](operating-rules.md), which carries the evidence
for each rule. Edit there, not here.

<!-- BEGIN OPERATING RULES SUMMARY -->
- **OR-1:** Build the tool; never leave a measurement in one-off code.
- **OR-2:** Run three to five sub-agents, at a model and thinking level matched to the
  task.
- **OR-3:** Never wait on a gate with nothing else in flight; run CI beside the
  research.
- **OR-4:** Take the next slice from the handoff, not from the backlog.
- **OR-5:** Declare the workflow entry point before beginning.
- **OR-6:** Plan multi-hour work in slices before starting it, as parallel lanes with
  disjoint deliverables.
- **OR-7:** Run the documentation guidelines pass at block boundaries.
- **OR-8:** A self-declared budget is not a stop condition.
- **OR-9:** A pull request leads with what the branch cost.
- **OR-10:** Treat matched agent and host handoffs as continuation, not a reset.
- **OR-11:** Close an agenda through disposition and reprioritization.
- **OR-12:** One block in four to eight is an efficiency block, and the record says
  which.
- **OR-13:** Every fast check runs in CI; only the unavoidably slow ones leave.
- **OR-14:** A development cycle is never artificially slow.
- **OR-15:** Outcome over ceremony, and process is revised on a cadence rather than on
  irritation.
- **OR-16:** Use Git for repository integrity; reserve checksums for real trust
  boundaries.
- **OR-17:** Every routine gate has a wall ceiling, and anything above it is selected on
  purpose.
<!-- END OPERATING RULES SUMMARY -->

## Build & Test

**This project runs Python 3.14. Never invoke the `python3` on `PATH`** — it is older,
and the two interpreters disagree about what parses.
Run `uv run --frozen ...` from `packing/`, or `packing/.venv/bin/python3` directly.

The trap this closes is specific and has caught four sub-agents:
[PEP 758](https://peps.python.org/pep-0758/) makes `except A, B:` valid without
parentheses, so files using it parse under the project interpreter and are reported as
syntax errors by anything older.
`sqpack/assurance.py` and `sqpack/contacts.py` both contain the construct and both are
correct. A report that they do not parse is a report that the wrong interpreter was used
— see [`D-397`](defects.md), and `OR-2` for the three occurrences before it.

### A fresh clone

Five things have to be true before any gate can run, and the gate checks none of them.
A remote-session clone on 2026-09-22 had all five false, and the edit tier came back
three steps red with every message pointing somewhere other than the cause.

```bash
curl -LsSf https://astral.sh/uv/install.sh | UV_INSTALL_DIR="$HOME/.local/bin" sh
uv python install 3.14.7                  # what packing/.python-version pins
git fetch --unshallow                     # remote-session clones arrive shallow
git submodule update --init --recursive
make hooks-install                        # npm ci, then the git hooks
```

Run `python3 -m devtools.check_bootstrap` from `packing/` to see which of the five are
already true; it prints the remedy for each one that is not, repairs nothing, and runs
under whatever `python3` the machine has, because a missing project interpreter is one
of the five things it reports.

Why each, since the failure it prevents never appears where it is caused:

- **uv before the interpreter.** uv’s download index is compiled in, so an old uv cannot
  install a new CPython at all.
  0.8.17 stops at `cpython-3.14.0rc2` and answers `uv python install 3.14.7` with “No
  download found for request”; `uv sync` then reports “No interpreter found for Python
  3.14.7”, which reads as a complaint about this repository.
  `uv self update` is not the way out — it asks the GitHub releases API and is
  rate-limited from a session proxy.
- **History before provenance.** A shallow clone makes `check_provenance` refuse
  recorded engine commits and leaves `check_session_gate` unable to decide the ancestry
  of a declared gate commit.
  Neither is a fact about the record.
- **The submodule before `uv sync`.** `vendor/kpress` is a declared workspace member, so
  an unchecked-out submodule makes uv report that it “does not appear to be a Python
  project”.
- **`npm ci` before the browser floor**, which asks for pinned binaries under
  `node_modules/.bin` rather than for anything on `PATH`.

The repository is mostly prose.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jlevy/squares](https://github.com/jlevy/squares) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
