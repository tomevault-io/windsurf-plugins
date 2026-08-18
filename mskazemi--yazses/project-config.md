---
trigger: always_on
description: A short, machine-readable brief for AI coding assistants working in this repository.
---

# AGENTS.md — project conventions for coding agents

A short, machine-readable brief for AI coding assistants working in this repository.
Humans should read [CONTRIBUTING.md](.github/CONTRIBUTING.md) instead; this file exists so an
agent-assisted contribution arrives correct on the first try.

**This file is canonical for every tool** — Codex, Claude Code, Gemini CLI, Cursor,
Copilot, Aider and anything else. There is deliberately no per-tool instruction file in
this repository: three copies of the setup commands become three different sets of setup
commands. If your tool looks for its own filename and finds nothing, read this one.

The human who opens the PR is responsible for it. Do not open a PR the author has not read.

## What this project is

YazSes is a cross-platform, **fully offline** hold-to-talk voice dictation daemon for Linux,
macOS, and Windows, plus offline file transcription (`yazses transcribe`) and whole-meeting
capture (`yazses meeting`). Speech-to-text is `faster-whisper` on CPU (int8). Python 3.11+,
managed with `uv`.

## Setup and gates

```sh
uv sync
uv run python -m pytest tests/ -v   # tests  (note: python -m pytest, not `uv run pytest`)
uv run ruff check src tests scripts # lint — same targets CI uses
uv run mypy src                     # types — clean today, advisory, see below
```

**pytest and ruff must be green.** Run them before claiming anything works — do not infer
success from a change looking correct. The suite is offline and mocks audio and model
layers, so no microphone, network, or Whisper model download is required.

**mypy is clean and advisory.** `uv run mypy src` currently reports **no issues across 433
source files**, so if it reports an error, you almost certainly just introduced it — fix it
rather than reporting it as pre-existing. It is not a CI gate (only `ruff` and `pytest`
are), but do not leave the count above zero.

The `[tool.mypy]` section in `pyproject.toml` silences the imports of optional backends a
base install deliberately omits. Those are absent by design, not bugs — do not "fix" them.

If you change any CLI command, flag, or config key, regenerate the reference docs or the
sync test will fail:

```sh
uv run python scripts/gen-docs.py   # rewrites docs/features.md, configuration.md, command-index.md
```

Useful narrower runs:

```sh
uv run python -m pytest tests/test_foo.py -v
uv run python -m pytest -k "pattern"
```

## Non-negotiable project rules

1. **Nothing leaves the machine.** Do not add network calls, telemetry, analytics, crash
   reporting, or a cloud API dependency. Offline-first is the product, not a preference.
   This is **enforced**: `tests/test_egress_inventory.py` fails the build when a module
   under `src/yazses/` gains an outbound primitive without being registered and classified
   in [ADR-019](design/adr/adr-019-egress-inventory-and-escalation.md). Seven paths exist
   and exactly **two** can transmit what the user said. If your change needs an eighth,
   read the ADR first — the escalation rules are written down, and three categories
   (voiceprints, the learning corpus, third-party capture) may never leave at all.
2. **New features ship off by default.** Add a config section in `src/yazses/config.py`
   with `enabled = False` (or an equivalent dormant default) so an existing install is
   unchanged until the user opts in.
3. **Heavy dependencies are optional extras.** Anything large (torch, ONNX runtimes, LLM
   runtimes, Qt) goes in `[project.optional-dependencies]` in `pyproject.toml` and is
   **imported lazily**, inside the function that needs it — never at module top level.
   A fresh install with no extras must import and run.
4. **Use the latest stable version** of any dependency you add, with a `>=` floor.
5. **Keep the pure logic pure.** Business logic goes in dependency-free modules that are
   unit-tested directly; heavy backends are injected. Follow the existing split — e.g.
   `meeting/segmenter.py` (pure) vs `meeting/silero_vad.py` (backend).
6. **Platform code goes behind the Protocols** in `src/yazses/platform/base.py`. To add an
   OS, implement every Protocol under `src/yazses/platform/<os>/` and register it in
   `platform/factory.py`.
7. **Tests come with the change**, in the same PR. New behaviour without a test is not done.
8. **No attribution lines** in commit messages — no `Co-Authored-By` trailers and no
   "generated with" footers.
9. **A guard is judged on how rarely it fires.** `cmdsafety`, `checkdigit` and the
   no-text-target guard all interrupt the user. One that fires on a house number teaches
   people to dismiss it, and a dismissed guard costs attention and catches nothing — so
   hold only on a *specific, checkable* signal, never a heuristic, and make the release
   one word. See [ADR-021](design/adr/adr-021-invest-in-error-cost.md).
10. **Cite the landing page, not the file.** No PDF is committed (`.gitignore` and the
    pre-commit hook both block it), and a `.pdf` URL is the wrong citation even though
    linking is not redistribution: it skips the page carrying the version, licence and
    DOI, and rots when the author reorganises their site.
    `tests/test_citation_hygiene.py` enforces both.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MSKazemi/yazses](https://github.com/MSKazemi/yazses) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
