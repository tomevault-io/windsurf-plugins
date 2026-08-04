---
trigger: always_on
description: This file gives Claude Code and other coding agents the current operating guidance for this repository.
---

# AGENT.md

This file gives Claude Code and other coding agents the current operating guidance for this repository.

## First Principles

1. **Read the relevant docs before touching a subsystem.** The `example/doc/` folder contains subsystem notes. Use these as maintenance references, but verify against code when they disagree.
2. **Check runtime overrides.** Defaults live in `station/constants.py`, but `station_data/constant_config.yaml` overrides them at import time. Always inspect both when behavior depends on configuration.
3. **Verify names and signatures before calling or editing.** Use `rg` to find definitions and call sites. Do not infer parameter names, constants, action names, or YAML fields from memory.
4. **Treat `station_data/` as live station state.** Reading is allowed for debugging. Ask the user before modifying real station data unless the request explicitly asks for that modification.
5. **Keep disposable local probes in `/tmp`.** Use `tests/test_*.py`, `tests/debug_*.py`, or `tests/analysis_*.py` only for files that should be visible to git. Use `/tmp` directly for generated API/debug snapshots, Sage/CAS probes, and other throwaway local test output.
6. **Use repository file helpers for persistent station data.** Use `station/file_io_utils.py` for atomic YAML/text writes and safe directory creation.

## Coding Agent Checklist

Before making code changes:

1. Run `git status --short` and identify unrelated user changes.
2. Read this file, then read the subsystem doc listed in the Documentation Map.
3. Inspect `station_data/constant_config.yaml` when configuration affects the issue.
4. Use `rg` to find the live implementation, constants, tests, and call sites.
5. Decide which files are code, docs, tests, or live station state before editing.

While changing code:

1. Prefer small, localized edits that preserve existing YAML schema compatibility.
2. Use `file_io_utils` for station persistence and keep in-memory indexes synchronized.
3. Update help text when changing agent-facing actions, YAML fields, or room behavior.
4. Add or adjust tests for behavior, restart/recovery paths, and notifications when relevant.
5. Do not change `.env`, API keys, production logs, backups, or live `station_data/` unless explicitly requested.

Before finishing:

1. Run the most relevant tests from the Verification Matrix below.
2. Re-run `git status --short`.
3. Report changed files, tests run, and any untested risk.

## Current Project Snapshot

- Package/version: `station` version `1.5.0` in `setup.py`, `station/__init__.py`, `README.md`, and `CITATION.cff`.
- Main app: `python -m web_interface.app`.
- Default data root: `./station_data`.
- Default research task layout: `station_data/rooms/research/`.
- Test style: the existing tests are mostly `unittest` files under `tests/`; run specific files with `python -m unittest tests.test_name`.
- Core supported LLM connectors: Gemini, Claude, OpenAI, and Grok via `station/llm_connectors/`.
- Current model presets are in `station/llm_connectors/model_presets.yaml`.

## Setup And Run

```bash
conda create -y -n station python=3.11
conda activate station
pip install -e .
python -m web_interface.app
```

Useful optional setup:

```bash
sudo apt install ripgrep
bash scripts/setup_theory.sh
```

Use the Theory setup only when working with `THEORY_ROOM_ENABLED` or Lean verification.

## Secrets And Local State

Sensitive or machine-local files can exist in this repo checkout. Treat these as read-only unless the user explicitly asks otherwise:

- `.env`
- `station_data/`
- `backup/`
- `deployment/`
- `worker_monitor.log`
- `example_private/`

API keys are read from environment variables such as `GOOGLE_API_KEY`, `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, and `XAI_API_KEY`. Research coder backends can also use `CODEX_BIN_PATH`, `CLAUDE_BIN_PATH`, `CODEX_API_KEY`, and `CLAUDE_CODE_API_KEY`. Do not print, copy, or rewrite secrets.

For live-station debugging, `deployment/error.log` is often useful, but it accumulates across station restarts. Do not run a blind global search across the whole file. First locate the most recent `Station initialized.` line, then inspect or search from that point onward.

## Documentation Map

- `example/doc/SYNC_MODES.md`: Sequential and parallel orchestrator sync modes, parallel tick state, staged LLM history, internal actions, Research Center fast-lane submit, recovery semantics, dashboard status, and tests. Read this before changing `SYNC_MODE`, `station_runner.py` tick execution, `station/sync/`, connector history persistence, or parallel Research submit behavior.
- `example/doc/API_BACKUP.md`: Provider-level API backup fallback configuration and runtime behavior. Read this before changing `station/runtime_api_config.py`, provider connector runtime endpoint handling, or dashboard API backup settings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dualverse-ai/station](https://github.com/dualverse-ai/station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
