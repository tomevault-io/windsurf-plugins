---
trigger: always_on
description: A short orientation file for an LLM working in this repo. Skim
---

# Notes for LLM contributors

A short orientation file for an LLM working in this repo. Skim
before making changes; keep edits consistent with what's described
here. Read [README.md](README.md) for the user-facing intro and
[CONTRIBUTING.md](CONTRIBUTING.md) for the human-facing
contributor guide.

## What this project is

`uiprotect` is an unofficial async Python client + CLI for the
UniFi Protect surveillance NVR. It is the library that the Home
Assistant `unifiprotect` integration is built on top of, and it
is the de-facto Python binding for talking to a UniFi Protect
console — Ubiquiti does not publish an official one. Public API
is exported from the top-level `uiprotect` package; the main
entry point is `uiprotect.ProtectApiClient` in `src/uiprotect/api.py`.

There is no upstream protocol owner. The wire format is whatever
the UniFi Protect server in front of you serves: a partially
documented REST API plus a binary WebSocket update stream. Both
shift across firmware releases, and the test corpus under
`tests/sample_data/` is the closest thing to a reference. Behaviour
changes that depend on a specific Protect firmware version should
say so in the commit message.

The project was forked from `pyunifiprotect` after that project
relicensed away from MIT; staying MIT-licensed is a hard
requirement so the Home Assistant core integration can keep
depending on it.

## Code style

- **Docstrings: terse, default to single-line.** A docstring is
  the function's _contract_, not its narrative. Almost every
  docstring should be one line — `"""Summary."""` — describing
  what the function does and what the caller can pass. Multi-line
  is the exception, only justified when there is non-obvious
  caller-visible behaviour the type signature and parameter names
  don't already convey. Ruff is configured with most of the `D`
  family disabled (`D100`–`D107`, `D2xx`, `D4xx`) precisely so
  that missing docstrings on internal helpers don't flag — only
  add one when it earns its keep.

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
  subtle invariant, a workaround for a specific Protect firmware
  quirk, behaviour that would surprise a reader. Firmware-version
  citations are useful when the reason for a branch is "this
  server returns the field, that one doesn't" — leave those in.
  If removing the comment wouldn't confuse a future reader, don't
  write it.

  **Don't remove existing comments** unless the code they
  describe is gone — the original author left them for a reason,
  often a specific Protect firmware regression.

- **Don't pad commits, docstrings, or comments with cross-
  references** to old codepaths or issue numbers unless there's
  a clear reason a future reader needs that link.

- **Method order**: public API at the top, private helpers
  (`_underscore_prefixed`) at the bottom. The supported surface
  is what `src/uiprotect/__init__.py` re-exports plus the public
  attributes on `ProtectApiClient`, `Bootstrap`, and the device /
  NVR model classes. Anything else is internal and can change
  between releases.

- **Line length**: 88 (ruff `line-length = 88`, not the more
  common 100/110). `requires-python = ">=3.11"`,
  `target-version = "py311"` for ruff; pyupgrade runs `--py311-plus`.

- **Imports**: ruff/isort sorted, `known_first_party = ["uiprotect", "tests"]`.
  Prefer `from __future__ import annotations` in new modules so
  modern type syntax works without runtime forward-reference
  juggling, especially given the pydantic model graph.

- **Pydantic v2 only.** Models live under `src/uiprotect/data/`
  and are built on pydantic v2 (`pydantic >= 2.13.4`). Do not
  reintroduce v1 idioms (`.dict()`, `Config` inner class,
  `validator` decorator) when touching this code — match the v2
  style already present (`model_dump()`, `model_config`,
  `field_validator`). The pinned-floor on pydantic is deliberate;
  do not loosen it without checking what changed across versions.

- **`orjson` for serialisation.** The codebase uses `orjson`
  everywhere it serialises or parses Protect payloads. Don't
  reach for stdlib `json` in hot paths; match the existing
  `orjson.loads` / `orjson.dumps` usage.

- **Async-only public surface.** UniFi Protect itself is async,
  and so is this library. New public functions on
  `ProtectApiClient` and friends should be `async def`. Don't
  introduce sync wrappers around async I/O; callers that need
  sync should drive the event loop themselves.

- **Typed strictly.** mypy runs with `disallow_untyped_defs`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uilibs/uiprotect](https://github.com/uilibs/uiprotect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
