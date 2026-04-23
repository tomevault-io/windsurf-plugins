---
trigger: always_on
description: > Guidelines for AI coding agents working in this Python codebase.
---

# AGENTS.md — markdown_web_browser

> Guidelines for AI coding agents working in this Python codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Python & uv

We use **uv** for everything in this project (env management, installs, running tools). **Never** call `pip`, `poetry`, `conda`, or ad-hoc `python -m venv`.

- **Runtime:** Python 3.13 only — no backward-compatibility shims
- **Packaging:** `pyproject.toml` exclusively; never introduce `requirements*.txt` or duplicate config files
- **Build system:** setuptools >= 70.0
- **Unsafe code:** N/A (Python)

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `fastapi` + `uvicorn` + `granian` | Web framework and ASGI servers |
| `playwright` | Browser automation (Chrome for Testing channel, pinned) |
| `pyvips` | Image tiling, resizing, PNG encode (primary image library) |
| `pillow` | Edge image format fallback only |
| `python-decouple` | Configuration via `.env` (exclusive config loader) |
| `pydantic` | Data validation and schemas |
| `httpx[http2]` | HTTP client for OCR and external APIs |
| `sqlmodel` + `sqlite-vec` | Database ORM + vector embeddings |
| `zstandard` | Compression |
| `mistune` | Markdown parsing |
| `prometheus-client` + `prometheus-fastapi-instrumentator` | Metrics and instrumentation |
| `structlog` | Structured logging |
| `typer` + `rich` | CLI framework and console output |
| `beautifulsoup4` | HTML parsing (DOM link extraction) |
| `numpy` | Numerical operations |
| `arq` + `redis` | Background job queue |
| `jinja2` | HTML templating |
| `psutil` | System/hardware introspection |

### Optional Dependency Groups

| Group | Packages | Purpose |
|-------|----------|---------|
| `local-ocr` | `vllm`, `sglang`, `olmocr` | Local OCR inference via vLLM/SGLang servers |
| `observability` | `opentelemetry-sdk`, `opentelemetry-exporter-otlp` | OpenTelemetry tracing |
| `dev` | `pyoxipng`, `pytest`, `pytest-asyncio`, `ruff` | Testing and linting |

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself

### No File Proliferation

If you want to change something or add a feature, **revise existing code files in place**.

**NEVER** create variations like:
- `capture_v2.py`
- `main_improved.py`
- `ocr_client_enhanced.py`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/markdown_web_browser](https://github.com/Dicklesworthstone/markdown_web_browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
