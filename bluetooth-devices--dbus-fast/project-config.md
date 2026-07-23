---
trigger: always_on
description: A short orientation file for an LLM working in this repo. Skim
---

# Notes for LLM contributors

A short orientation file for an LLM working in this repo. Skim
before making changes; keep edits consistent with what's described
here. Read [README.md](README.md) for the user-facing intro and
[CONTRIBUTING.md](CONTRIBUTING.md) for the human contributor flow.

## What this project is

`dbus-fast` is a Python client/server library for
[D-Bus](https://dbus.freedesktop.org/), targeting Linux desktop and
mobile environments. It's a performance-focused fork of
[`python-dbus-next`](https://github.com/altdesktop/python-dbus-next)
and powers D-Bus-side integrations in
[Home Assistant](https://www.home-assistant.io/) and the
[bluetooth-devices](https://github.com/bluetooth-devices/) ecosystem
(BlueZ, BleakDBus, etc.).

Two IO backends are shipped from the same wire-level core:

- `dbus_fast.aio` — asyncio (`MessageBus`, `MessageReader`,
  `ProxyObject`). What most callers use.
- `dbus_fast.glib` — GLib main-loop bindings, for GTK/GObject
  callers.

Hot paths are Cythonized at build time for throughput. They keep
working as pure Python — `SKIP_CYTHON=1` disables the extension
build — but production wheels ship compiled and CodSpeed
benchmarks track that path. See _Build conventions_ below.

## Code style

- **Docstrings: terse, default to single-line.** A docstring is
  the function's _contract_, not its narrative. Almost every
  docstring should be one line — `"""Summary."""` — describing
  what the function does and what the caller can pass. Multi-line
  is the exception, only justified when there is non-obvious
  caller-visible behaviour the type signature and parameter names
  don't already convey.

  **What does NOT belong in docstrings or comments:**
  - Rationale / motivation / "why we used to do X" — that's the
    PR description and the commit message. Git already remembers.
  - Cross-references to issue numbers ("closes #N", "follow-up
    to #M") — the PR body carries those.
  - Restatement of the function body in prose. If the next line
    of the docstring is just describing what the next line of
    code does, delete the docstring line.
  - Test docstrings retelling the production-side story. A test
    docstring should name what the test pins, in one sentence —
    not re-explain the bug, the fix, or the surrounding flow.

- **Comments**: same bar. Default to writing no comments. Add
  one only when the _why_ is non-obvious: a hidden constraint, a
  subtle invariant, a workaround for a specific bug, behaviour
  that would surprise a reader. If removing the comment wouldn't
  confuse a future reader, don't write it.

  **Don't remove existing comments** unless the code they
  describe is gone — the original author left them for a reason.
  Comments around Cython-specific patterns (`if cython.compiled:`,
  branch hints, manual inlines) frequently exist _because_
  removing them regressed a benchmark or broke the C build.

- **Don't pad commits, docstrings, or comments with cross-
  references** to old codepaths or issue numbers unless there's
  a clear reason a future reader needs that link.

- **D-Bus type annotations: no signature strings.** Since v4.0.0
  the supported form is the `Annotated` aliases in
  `dbus_fast.annotations` (`DBusInt32`, `DBusUInt16`, `DBusStr`,
  …) or `Annotated[..., DBusSignature(sig)]` for a runtime
  signature. The old string form (`def m(self) -> "i":`) is
  deprecated. No runtime `DeprecationWarning` is emitted, so
  review feedback is the only signal — don't reintroduce it in
  code or docs.

- **Method order**: public API at the top, private helpers
  (`_underscore_prefixed`) at the bottom.

- **Line length**: 88 (ruff default with overrides — see
  `pyproject.toml`). `python_requires = ">=3.10"`,
  `target-version = "py310"` for ruff and `--py310-plus` for
  pyupgrade. Don't introduce 3.11+-only syntax.

- **Imports**: ruff/isort sorted (`profile = "black"`,
  `known_first_party = ["dbus_fast", "tests"]`). Prefer absolute
  imports rooted at `dbus_fast.*`.

- **Generated artefacts are not checked in to source paths.**
  Cython produces `*.c`, `*.html`, and `*.so` siblings of each
  `.py` listed in `TO_CYTHONIZE`; `*.c` is excluded from the
  wheel via `pyproject.toml`'s `exclude` list and `*.html` /
  `*.so` are build outputs — don't commit them.

## Commit / PR conventions

- **Conventional Commits, lowercase subject.** Repo is squash-
  merge only with `squash_merge_commit_title = PR_TITLE`, so the
  PR title — not the individual branch commits — becomes the
  subject of the commit that lands on `main`. The
  `pr-title.yml` workflow runs
  `amannn/action-semantic-pull-request` against the title; the
  `commitizen` pre-commit hook gives local feedback on per-commit
  subjects but CI no longer re-checks them, because the squash
  discards them anyway. Accepted types: `build`, `chore`, `ci`,
  `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`,
  `test`. Scopes are optional. The subject (text after
  `type(scope):`) must start lowercase. Examples that pass:
  - `feat: add async context manager to MessageBus`
  - `fix(unmarshaller): handle empty arrays at end of frame`
  - `perf!: drop python 3.9 support`
- **Releases are commit-driven.** `python-semantic-release` reads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bluetooth-Devices/dbus-fast](https://github.com/Bluetooth-Devices/dbus-fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
