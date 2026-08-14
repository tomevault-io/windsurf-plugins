---
trigger: always_on
description: This file provides guidance to coding agents (Codex, Cursor, Aider, and any tool that reads `AGENTS.md`) when working with code in this repository. The content mirrors `CLAUDE.md`; either file is authoritative.
---

# AGENTS.md

This file provides guidance to coding agents (Codex, Cursor, Aider, and any tool that reads `AGENTS.md`) when working with code in this repository. The content mirrors `CLAUDE.md`; either file is authoritative.

## Development commands

```bash
python -m pip install -e ".[dev]"      # install with dev deps (pytest)
pytest                                  # full suite
pytest tests/workbench/ -v              # a single directory
pytest tests/test_cli.py::test_name     # a single test
```

Python 3.10+ is required (CI matrix: 3.10–3.13, `.github/workflows/test.yml`).

Frontend (only when touching the workbench UI — the PyPI wheel ships the pre-built assets):

```bash
cd clawjournal/web/frontend && npm install && npm run build
```

The CI `smoke` job verifies that the built wheel contains `clawjournal/web/frontend/dist/index.html`. Touching the frontend without rebuilding will silently regress the wheel.

A running `clawjournal serve` pins the bundle it captured at startup, so a rebuild is invisible to it until you restart the daemon — `serve --reload` stays disk-backed and picks rebuilds up on reload.

CLI entry point: `clawjournal = clawjournal.cli:main` (see `pyproject.toml`). README.md has the full user-facing command reference.

## Architecture

ClawJournal is a local-first tool that scans coding-agent session logs, indexes them into SQLite, runs a findings (secrets + PII) pipeline, and lets the user triage, score, redact, and export/share. Everything defaults to local; sharing is a separate opt-in step.

### Package layout (`clawjournal/`)

- `cli.py` — single-file argparse CLI (~4k lines) dispatching all subcommands.
- `config.py` — `~/.clawjournal/config.json` read/write, TypedDict schema, migrations. `CONFIG_DIR` (`~/.clawjournal/`) is the well-known install root used by the rest of the code.
- `selfupdate.py` — hourly, throttled background ff-only auto-update of the editable checkout; after a pull that changes deps/frontend/scanner pins it reruns the installer automatically (`pending_reinstall.json` record is the failure-fallback notice). `selfupdate --check/--reinstall/--clear-pending` are the manual surface; `scripts/install.{sh,ps1}` also self-sync a clean-main checkout before installing. Before the updater starts, a normal `serve` process captures an immutable in-memory frontend snapshot matching its imported backend; once the update is reconciled, the daemon gracefully re-execs after requests and background work are idle (request admission freezes before listener shutdown; disabled under `--reload`) and the new process captures the new frontend/backend pair together.
- `paths.py` — atomic bootstrap of `~/.clawjournal/hash_salt` (salts findings' `entity_hash`) and `~/.clawjournal/api_token` (bearer token for the loopback daemon API). Write-then-link pattern prevents races between CLI and daemon on a fresh install.
- `parsing/` — per-agent source discovery + normalization. `parser.py` defines each agent's on-disk location (`CLAUDE_DIR`, `CODEX_DIR`, `GEMINI_DIR`, `OPENCODE_DIR`, `OPENCLAW_DIR`, `KIMI_DIR`, `CURSOR_DIR`, `COPILOT_DIR`, `AIDER_*`, `CUSTOM_DIR`, `LOCAL_AGENT_DIR` for Claude Desktop) and converts raw logs to a shared session shape. `segmenter.py` handles session boundary logic.
- `capture/` — incremental ingest adapter (JSONL cursors, change detection, discovery) used by the background scanner in the daemon.
- `redaction/` — layered stages:
  - `anonymizer.py` anonymizes home-dir paths + usernames (always, including before anything is sent to an AI backend).
  - `secrets.py` regex secrets detection with entropy heuristics (`_has_mixed_char_types`, `_shannon_entropy`).
  - `pii.py` optional AI-assisted PII review (`review_session_pii*`), producing `findings` and applying them.
  - `betterleaks.py` / `trufflehog.py` — subprocess wrappers for the two share-gate scanners (managed installs in `betterleaks_install.py`/`trufflehog_install.py`, pinned + checksum-verified; pin CI in `scripts/check_*_pin.py`). Betterleaks is also a default findings engine; `betterleaks.toml` is the bundled gate config (placeholder allowlist only — tier logic stays in Python).
  - `scan_policy.py` — the tier policy (`classify`, `GateReport`): per-finding block/review/redact/warn decisions consumed by the share gate and preview gates.
- `findings.py` — substrate for the scan-time findings pipeline (hashed entity references; plaintext is never persisted — salt lives in `paths.py`).
- `scoring/` — judge-backed 1–5 quality scoring. `backends.py` picks a backend (Claude CLI / `codex exec` / other), `scoring.py` orchestrates, `badges.py` computes outcome/value/risk badges used in the index, `depth.py`/`insights.py` support card generation.
- `workbench/`
  - `index.py` — SQLite + FTS5 schema + all queries. `SECURITY_SCHEMA_VERSION` is bumped with gated migrations — don't rewrite historical migrations, add a new one.
  - `daemon.py` — `clawjournal serve` HTTP API (loopback, bearer-token gated) + background scanner. Serves the Vite build under `web/frontend/dist/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rayward-external/clawjournal](https://github.com/rayward-external/clawjournal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
