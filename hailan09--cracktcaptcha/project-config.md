---
trigger: always_on
description: Guide for AI coding agents working on this repository. Follows the [agentsmd.org](https://agentsmd.org) convention.
---

# AGENTS.md

Guide for AI coding agents working on this repository. Follows the [agentsmd.org](https://agentsmd.org) convention.

## 1. Project Overview

`crack-tcaptcha` is a pure-HTTP, Python-based automated solver for Tencent's
T-Sec TCaptcha (TCaptcha 2.0, `turing.captcha.qcloud.com`). It supports four
challenge types (`slider`, `icon_click`, `word_click`, `image_select`) and
does **not** drive a real browser — it replays the official JavaScript
fingerprint / behavior collector (`TDC.js`) inside a Node.js + jsdom
subprocess and speaks the captcha HTTP protocol directly with Chrome
TLS/HTTP2 fingerprint emulation (via [`wreq`](https://github.com/0x676e67/wreq-python)).

See `docs/` for user-facing documentation and `docs/architecture.md` for the
layered architecture diagram.

## 2. Build / Test / Run Commands

Python >= 3.10, `uv` is the canonical package manager.

```bash
# Install deps (no extras)
uv sync

# Install with optional extras
uv sync --extra icon-click   # ddddocr + onnxruntime (icon_click pipeline)
uv sync --extra word-click   # onnxruntime + opencv-headless + ddddocr (word_click pipeline, local YOLO+Siamese)
uv sync --extra dev          # pytest, respx, ruff, hypothesis
uv sync --extra docs         # mkdocs-material

# Node.js side (TDC.js bridge) — required the first time
cd src/crack_tcaptcha/tdc/js && npm install

# Tests
uv run pytest                             # full suite (offline)
uv run pytest -m "not network"            # default (network tests already marked)
uv run pytest tests/pipelines/ -q         # a single directory

# Lint / format
uv run ruff check .
uv run ruff format .

# CLI — one-shot
uv run crack-tcaptcha solve --appid YOUR_APPID --entry-url https://your-site.example/login

# CLI — long-running HTTP service (recommended for repeated use; models load once)
uv run crack-tcaptcha serve --port 9991 --workers 4
#   POST http://127.0.0.1:9991/solve  {"appid":"YOUR_APPID","retries":3}
#   GET  http://127.0.0.1:9991/health
#   set TCAPTCHA_SERVE_SK to require an X-SK header.

# Docs
uv run mkdocs serve
```

## 3. Architecture Map

```
src/crack_tcaptcha/
├── __init__.py          # public API: solve()
├── captcha_type.py      # pure-function classifier (dyn_show_info → type)
├── cli.py               # argparse entry point (solve / serve subcommands)
├── server.py            # long-running HTTP service (stdlib http.server)
├── client.py            # HTTP three-phase + JSONP unwrap (wreq Chrome emulation)
├── exceptions.py        # NetworkError, SolveError, PowError, TDCError
├── models.py            # pydantic models for prehandle / verify responses
├── pow.py               # MD5 PoW solver with calc_time shaping
├── settings.py          # pydantic-settings (TCAPTCHA_* env vars, .env)
├── trajectory.py        # slide/click trajectory generation
├── pipelines/
│   ├── _common.py       # run_async, finish_with_verify (shared tail)
│   ├── slide.py         # NCC template match
│   ├── icon_click.py    # ddddocr detect + template match
│   ├── word_click.py    # YOLO detect + Siamese match (local ONNX); ddddocr OCR fallback
│   └── image_select.py  # LLM region matching
├── solvers/
│   ├── ort_provider.py  # ONNX Runtime execution-provider selection
│   ├── word_ocr.py      # YOLOv8 + Siamese solver for word_click (fast path)
│   ├── llm_vision.py    # OpenAI-compatible vision client (image_select only)
│   └── models/          # bundled ONNX models + font.ttf (force-included in wheel)
└── tdc/
    ├── provider.py      # TDCProvider Protocol (DI point)
    ├── nodejs_jsdom.py  # Node.js subprocess implementation
    └── js/              # tdc_executor.js + vendored tdc.js
```

Dependency direction is strictly top-down: `pipelines/` depends on
`solvers/`, `tdc/`, `client.py`, `pow.py`, `trajectory.py`. `solvers/` and
`tdc/` are independent of each other and must not import from `pipelines/`.
`server.py` depends on `__init__.solve` and may trigger `solvers/word_ocr.warmup`
at startup — it must not import from `pipelines/` directly.

## 4. Key Conventions

- **Type hints everywhere.** `from __future__ import annotations` at the top
  of every module. PEP 604 unions (`str | None`) are fine because Python 3.10+.
- **Config via pydantic-settings.** Don't read env vars directly; use
  `crack_tcaptcha.settings.settings`. New settings go in `settings.py` with
  a `TCAPTCHA_` prefix and sensible defaults.
- **Data models via pydantic v2.** Response shapes live in `models.py`; never
  pass raw dicts across module boundaries.
- **Logging over prints.** Use `log = logging.getLogger(__name__)` and
  log at INFO for pipeline milestones, DEBUG for inner workings, WARNING
  for recoverable failures. No `print()` in library code.
- **Exceptions.** Raise the typed exceptions in `exceptions.py`
  (`SolveError`, `NetworkError`, `PowError`, `TDCError`). `SolveError`
  specifically is the "this attempt failed, caller may retry" signal.
- **Line length 120** (ruff). `ruff.lint.select = ["E","F","I","UP","B","SIM"]`.
- **No new top-level deps without discussion.** Optional features go behind
  an `extras_require` group (see `pyproject.toml` `[project.optional-dependencies]`).

## 5. Gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hailan09/crackTCaptcha](https://github.com/hailan09/crackTCaptcha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
