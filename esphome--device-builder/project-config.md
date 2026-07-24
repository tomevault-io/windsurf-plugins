---
trigger: always_on
description: A short orientation file for an LLM working in this repo. Skim before
---

# Notes for Claude

A short orientation file for an LLM working in this repo. Skim before
making changes; keep edits consistent with what's described here. Read
[README.md](README.md) for the user-facing intro and
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the deep dive.

## What this project is

Backend for the new ESPHome Device Builder dashboard — a WebSocket API
server that replaces the legacy `esphome dashboard`. Single multiplexed
`/ws` endpoint, persistent firmware-job queue, mDNS + ping device
discovery, schema-driven component catalog, curated board catalog.
Frontend is a separate repo (`esphome/device-builder-frontend`) and
ships prebuilt inside our wheel.

**Issues filed here are often frontend bugs.** Users file every
dashboard bug against this backend repo, but the Visual Editor,
component-catalog rendering, and YAML-form UI all live in the
`esphome/device-builder-frontend` SPA. When triaging, check whether the
symptom is UI / editor behaviour before assuming a backend fix, and
trace both sides: the backend can parse and emit correctly while the
frontend's rendering or value-sync drops the data. A frontend fix is a
PR against `esphome/device-builder-frontend` (base `main`) that links
`fixes <this repo's issue URL>` cross-repo. Example: issue #1005 (dotted
`logger.logs` map keys lost in the Visual Editor) was a frontend
`data-field-key` round-trip bug, not a backend one.

Base functions are in late beta; remote / offload functions are in early
beta. Expect undocumented breaking changes until stable. Targeted to
land as an opt-in preview toggle in the official ESPHome container and
Home Assistant add-on.

## Code style

- **Docstrings: terse, default to single-line.** A docstring is the
  function's *contract*, not its narrative. Almost every docstring
  should be one line — `"""Summary."""`. Multi-line is the exception,
  justified only by genuinely non-obvious caller-visible behaviour the
  signature and parameter names don't convey. When multi-line is needed,
  put content on the line after `"""`:

  ```python
  def merge_component_yaml(...) -> str:
      """
      Render *component* and merge it into *existing* YAML.

      Platform-style components append under the existing ``<domain>:``.
      """
  ```

  Default target for new code: three lines between the `"""` markers.
  Longer is acceptable when the contract genuinely needs it (non-obvious
  priority orders, security-relevant fallbacks, an empirical anchor like
  an issue number proving a value matters, load-bearing ordering).

  **Never put in docstrings or comments:** rationale / motivation /
  "why we used to do X" (that's the PR/commit); issue-number cross-refs
  ("closes #N"); prose restatement of the function body; test docstrings
  retelling the production story (name what the test pins, in one
  sentence); "same shape as X / mirrors Y" framing.

- **Comments**: same bar. Default to none. Add one only when the *why*
  is non-obvious: a hidden constraint, subtle invariant, bug workaround,
  surprising behaviour. **Don't remove existing comments** unless the
  code they describe is gone.

- **Don't pad commits, docstrings, or comments with cross-references**
  to old codepaths or issue numbers unless a future reader needs the
  link.

- **Method order**: public API at the top, private helpers
  (`_underscore_prefixed`) at the bottom. Same for module-level
  functions in scripts.

- **Line length**: 100 (ruff). `target-version = "py312"`.

- **Imports**: ruff/isort sorted. `from __future__ import annotations`
  at the top of every module.

- **File size: 800-line soft cap.** When a Python module reaches ~800
  lines, plan a split before adding more. The cap exists because large
  modules degrade LLM context budget, slow human review, and accumulate
  invisible cross-concern coupling.

  The split pattern: replace `controllers/X.py` with a `controllers/X/`
  package containing `controller.py` (main class + public API) plus
  per-concern submodules (`controllers/X/foo.py`). `__init__.py`
  re-exports the controller class so existing
  `from .controllers.X import XController` callers keep working. See
  `controllers/devices/`, `controllers/firmware/`,
  `controllers/remote_build/` for the canonical shape.

  **State dataclass convention.** Group mutable *domain* state —
  anything a sibling module reads or writes — into a typed `XxxState`
  dataclass in `controllers/X/_state.py`; the controller owns
  `self.state: XxxState`; siblings reach through `controller.state.X`.
  Canonical: `OffloaderState`, `DevicesState`, `ReceiverState`.
  Controller-internal handles no sibling touches (`_unsub_job_completed`,
  `_pairings_store`, base infra like `_db` / `_listeners` / `_tasks`)
  stay on the controller even when reassigned across `start()` /
  `stop()`. The cut is cross-module data, not reassignment. Design with
  `XxxState` from PR 1 of a split — adding it later forces a cleanup arc
  (#795, #797). When `start()` / `stop()` repopulates a `state.X`
  dict/set something captured a bound method of (`.add`, `.__contains__`),
  use in-place `.clear()` + `.update(...)` rather than reassignment, or
  the captured method points at the original empty container forever

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esphome/device-builder](https://github.com/esphome/device-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
