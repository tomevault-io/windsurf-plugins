---
trigger: always_on
description: > User-facing layer of this uv workspace: the `palmimo_sdk` core
---

# AGENTS.md — Palmimo DevKit software

> User-facing layer of this uv workspace: the `palmimo_sdk` core
> (motion engine, public API, servo I/O) and the agent apps.

**A procedure has one home** — see [Documentation](#documentation) for which page
that is. This file holds the Python development rules agents need on every task
in this repository.

> **Scope:** the uv / Python rules below apply throughout this repository.

## Quick Reference

- Language: Python >=3.12
- Package manager: **uv** (NEVER pip)
- Build backend: hatchling
- Run from: the repository root
- Execute: `uv run python <script>` (NEVER bare `python`)
- Add deps: `uv add <package>` (NEVER `pip install`)
- Sync deps: `uv sync` (+ `uv sync --group dev` for ruff/mypy)
- Lint/format: `uv run ruff check .` / `uv run ruff format .`
- Type check: `uv run mypy` (bare — pyproject's `files` decides the target set, same as CI; passing `.` would skip it)

## Layout

```
packages/
  palmimo_sdk/                    -> Core SDK — the single user-facing window
    robot.py                   -> Palmimo facade (public API + connection lifecycle)
    engine.py                  -> MotionEngine (pure gait/IK, no I/O)
    kinematics.py              -> Shared leg kinematics
    io/base.py                 -> ServoDriver ABC (I/O boundary)
    io/dynamixel.py            -> DynamixelDriver (concrete, over Dynamixel bus)
    io/camera.py, io/display.py, io/microphone.py, io/speaker.py
                               -> peripheral backends the Palmimo facade owns
    tests/                     -> engine / robot / driver / kinematics tests
examples/
  agents/
    wakeword/                    -> Wake-word voice agent example (Silero VAD + Whisper STT + LLM tool-calling)
    companion/                   -> Always-on companion agent example (idle loop + speech/vision-driven responses, LLM tool-calling)
    openclaw/                    -> Connection kit for driving Palmimo from OpenClaw (self-hosted AI assistant) over the MCP server
scripts/                       -> Supported user diagnostics
tests/                         -> Tests owned by the tree, not by one package
  contracts/                   -> Doc placement, naming, and comment-language ratchets
  scripts/                     -> Tests for scripts/
```

> Anything built on top of the SDK consumes `palmimo_sdk` as its single window.
> The core never depends back on a package that builds on it.

## Architecture (read [doc/explanation/architecture.md](doc/explanation/architecture.md) for details)

The `palmimo_sdk` package is the core; everything else builds on its shared computation:

1. **palmimo_sdk.engine — MotionEngine** — Pure gait/IK computation. NEVER sends hardware commands.
2. **palmimo_sdk.robot — Palmimo** — Public facade. Maps string commands -> Motion enum; owns connection lifecycle.
3. **palmimo_sdk.io — ServoDriver / DynamixelDriver** — I/O boundary; adapts position dicts to the servo bus.

Anything built on the SDK delegates gait to MotionEngine and never opens a
hardware backend of its own.

Data flow: `robot.forward()` -> `robot.step()` -> MotionEngine computes -> returns `Dict[str, int]` (servo_name -> tick_value) -> ServoDriver sends to hardware (or compute-only in dry-run).

## Coding Rules

- SPDX headers (`# SPDX-License-Identifier: Apache-2.0`) apply only to
  `palmimo_sdk` package code (`packages/palmimo_sdk/palmimo_sdk/`, excluding
  its own `tests/`) — never to examples, tests, or scripts. Enforced by
  [tests/contracts/test_license_headers.py](tests/contracts/test_license_headers.py).
- Type hints on ALL function signatures
- Google-style docstrings
- Constants: UPPER_SNAKE_CASE / Classes: PascalCase / Functions: snake_case
- Each gait method only modifies servo positions — no side effects
- New motions MUST transition smoothly from/to neutral stance
- Tests: dry-run first (`--live` flag off), verify servo range 200-3900 (safe range)
- When changing palmimo_sdk (robot.py, engine.py, kinematics.py), update the co-located docs under `doc/` ([api-reference.md](doc/reference/api-reference.md), [architecture.md](doc/explanation/architecture.md), [motion-development-guide.md](doc/guides/motion-development-guide.md)) in the same commit

## Comments

- Everything in this tree ships to users, so the text explaining the code is
  **English only**: Python comments, docstrings, and the text a diagnostic
  carries (an `assert`'s message, a raised exception's arguments, a log
  record), `#` comments in the config files (`pyproject.toml`, YAML, shell,
  `.gitignore`, `.env` samples), Markdown prose, and the labels in
  `doc/images/*.drawio.svg`.
- **The tree carries no translation debt.** `JAPANESE_COMMENT_DEBT`
  ([tests/contracts/test_comment_language.py](tests/contracts/test_comment_language.py))
  is empty, and a test keeps it empty — translate a file rather than parking it
  on the list. Reopening the list for a staged migration means deleting that
  test in the same commit, so the decision is reviewed rather than assumed.
- Document *why*, not *what*. Rationale, alternatives weighed, and change
  history go in the commit message and PR description — not the source.
- The ratchet checks explanation, not output. Japanese in a string the product

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jizai-inc/palmimo-devkit](https://github.com/Jizai-inc/palmimo-devkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
