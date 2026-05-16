---
trigger: always_on
description: > **Project memory hub for AI coding agents.** Claude Code reads this natively. Other agents (Cursor, Codex, Gemini CLI, Aider) should read this as a top-level reference.
---

# CLAUDE.md

> **Project memory hub for AI coding agents.** Claude Code reads this natively. Other agents (Cursor, Codex, Gemini CLI, Aider) should read this as a top-level reference.

## What this project is

`gflow-cli` is an unofficial Python CLI that drives [Google Flow](https://labs.google/fx/tools/flow) (Veo image-to-video, Imagen text-to-image) from the terminal by reverse-engineering Flow's private REST API at `aisandbox-pa.googleapis.com`. Built for Google AI Ultra/Pro subscribers who want to spend their Flow credits via batch automation rather than the web UI.

**Same pattern as `edge-tts`:** a community SDK over a private cloud API.

## On every session start

1. Read **[docs/INDEX.md](docs/INDEX.md)** — routing layer for all project docs.
2. Read **[PLAN.md](PLAN.md)** — current phase, next concrete tasks, ADRs.
3. Read **[KNOWN_ISSUES.md](KNOWN_ISSUES.md)** — open issues to avoid re-discovering.
4. Check **[CHANGELOG.md](CHANGELOG.md) `[Unreleased]`** — what's recently shipped.

## Active phase

**Phase 4 — Hardening DONE (v0.4.0a2).** Per-worker Page pool, tenacity
retry/backoff with reCAPTCHA re-mint, RFC 9457 Problem Details error
taxonomy with per-class exit codes 3–7, structlog observability with
`error_raised` / `error_unhandled` events, and 12 pytest-bdd scenarios all
shipped. Documentation polish landed in v0.4.0a2. Next phase TBD — likely
Phase 5 public alpha on PyPI followed by Phase 6 operations history
(see `PLAN.md`).

## Architecture (skim)

> Note: the layered diagram below describes the **target** architecture
> (deferred per [PLAN.md ADR #2](PLAN.md#5-decision-log-adrs-in-miniature)).
> The current package layout is the simpler
> `src/gflow_cli/{api/, cli.py, cli_image.py, cli_video.py, _cli_helpers.py,
> auth.py, config.py, errors.py, observability.py, paths.py,
> profile_store.py}`. See
> [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full target shape.

```text
interfaces/   →  application/   →  domain/   ←  infrastructure/
   CLI            commands /          pure         FlowProvider,
                  queries /         business        Playwright,
                  handlers /          logic         filesystem
                  ports
```

Dependency rule: **`domain/` depends on nothing**. `application/` depends on `domain/` + ports (Protocols). `infrastructure/` implements the ports. `interfaces/` wires it together.

## CRITICAL rules — invariant, never violate

- **Never commit secrets.** `.gitignore` excludes `auth/`, `profile_*/`, `*.cookies.json`, `storage_state.json`, `secrets.json`, `.env`, `.env.local`, `.env.*.local`. If you see one of these staged, abort and tell the user.
- **Never add `Co-Authored-By: Claude` (or any AI co-author) to commit messages.** Author attribution is the human user's only.
- **Sessions belong outside the repo.** Default location is `$LOCALAPPDATA/gflow-cli/profile_*` (Windows) or `~/.local/share/gflow-cli/profile_*` (POSIX) via [`platformdirs`](https://github.com/platformdirs/platformdirs). Never store sessions in `/tmp`, `%TEMP%`, or anywhere the OS auto-reaps.
- **No raw `print()` and no `import logging` in `src/`** — use `structlog` for structured events, or Rich `console.print(...)` for user-facing output.
- **Domain layer is pure** — `src/gflow_cli/domain/*` must not import `application/`, `infrastructure/`, or `interfaces/`. No I/O, no frameworks.
- **Frozen dataclasses for value objects.** Aggregates may have controlled mutation methods, but VOs (AspectRatio, Prompt, JobId, ...) are immutable.
- **Async all the way down.** Handlers and providers are `async def`. CLI is the only place that calls `asyncio.run(...)`.
- **TDD is non-negotiable.** Red → Green → Refactor → Commit. Coverage floor: **80% overall**, **90% on `domain/` and `application/`**. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Coding conventions

- **Python 3.11+**, strict typing (`pyright --strict` on `src/gflow_cli/`), `from __future__ import annotations` at the top of every module.
- **`@dataclass(frozen=True)`** for value objects and DTOs. `Protocol` for ports.
- **`pathlib.Path`** everywhere — never raw strings for filesystem paths.
- **Click + Rich** for the CLI, **Playwright `page.request`** as the HTTP transport (auto-attaches Google session cookies), **`tenacity`** for retry/backoff, **`structlog`** for structured logs, **`pytest + pytest-bdd`** for tests.
- **Conventional Commits** for messages (`feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`, etc.). See examples in `git log`.
- **Short files, single responsibility.** ~200-400 lines typical, 800 max. Split into `package/<topic>.py` if growing.

## How to verify your work

Run all four BEFORE asking to commit:

```bash
uv run ruff check src tests          # lint
uv run ruff format --check src tests # formatting
uv run pyright src                   # types
uv run pytest -q --cov=gflow_cli      # tests + coverage
```

CI runs the same four on every push (see `.github/workflows/ci.yml`).

## Where to look

| I need to… | Read this |
|---|---|
| Understand the architecture | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ffroliva/gflow-cli](https://github.com/ffroliva/gflow-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
