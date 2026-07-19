---
trigger: always_on
description: **This repository** (`https://github.com/omarelkhal/aftertone`) is the **default project** for Cursor/agent work (spoken TTS, hooks, daemon). Do not confuse it with the upstream **`supertone-inc/supertonic`** clone unless the user is contributing to Supertonic itself.
---

## Aftertone — agent map

**This repository** (`https://github.com/omarelkhal/aftertone`) is the **default project** for Cursor/agent work (spoken TTS, hooks, daemon). Do not confuse it with the upstream **`supertone-inc/supertonic`** clone unless the user is contributing to Supertonic itself.

- **Goal:** post-reply **local TTS** for coding agents (Cursor today; Claude & Codex adapters tracked in [CONTRIBUTING.md](CONTRIBUTING.md)).
- **`.cursor/hooks.json`** — Cursor adapter; must include `"version": 1`.
- **`.cursor/hooks/`** — `speak_summary.sh`, `speak_summary.toml`, `README.md` (full TOML reference), `hook_payload_trace.sh`.
- **`py/`** — `tts_daemon.py`, `tts_daemon_ctl.py`, `speak_summary_prepare.py`, `speak_summary_toggle.py` (flip `enabled` in TOML), `speak_summary_config.py` (lang/speed/mode/voice), vendored `helper.py` (Supertonic), `tts_io.py`, `fetch_assets.py`, diagnostics.
- **`.cursor/commands/`** — **only user-facing way** to change spoken-TTS settings (`/aftertone-toggle`, `/aftertone-lang`, …). Each command is **one** `uv run --directory py python -m aftertone …` from the install root (`aftertone-install-dir`); agent must **not** plan or hand-edit TOML. Settings go through `aftertone set lang|speed|mode|voice|expression` (delegates to `speak_summary_config.py`); voice restarts the daemon by default.
- **`scripts/bootstrap.sh`** — `uv sync` + HF snapshot if ONNX dir missing.

## Commands

- One-line install: `curl -fsSL .../install.sh | bash -s -- --install-uv --start-daemon` → **`~/aftertone`** + **user hooks** in `~/.cursor/hooks.json` (default `--global`). Legacy per-project: `--no-global --into .`. See `scripts/install.sh`.
- **Uninstall:** `curl -fsSL .../scripts/uninstall.sh | bash` or `bash scripts/uninstall.sh` (Linux); `irm .../scripts/uninstall.ps1 | iex` or `powershell -File scripts\uninstall.ps1` (Windows) — stops daemon, removes user `.cursor` Aftertone hooks/commands/rule; deletes install dir unless `--keep-dir` / `-KeepDir`.
- Bootstrap: `bash scripts/bootstrap.sh` from repo root.
- Daemon: `cd py && uv run python tts_daemon_ctl.py start --repo-root ..`
- **User config:** slash `/aftertone-*` only ([`.cursor/commands/`](.cursor/commands/)). Agent runs **one** `python -m aftertone …` from the install root — no planning preamble, no bash `aftertone-root.sh`, no hand-edited TOML. **lang/speed/mode/voice** without a value: **AskQuestion** first; **voice** uses `aftertone set voice PRESET --ensure` (daemon restart is default).
- Tests: `cd py && uv sync && uv run pytest` — `py/tests/` (integration tests run `speak_summary_prepare.py` with temp TOML; unit tests cover helpers including quiet hours).
- `uv run` examples: `cd py` first, or `uv run --directory py …` from repo root.

## Env

- **`AFTERTONE_REPO`** / **`AFTERTONE_INSTALL_DIR`** — install root (`~/aftertone` by default; `~/.cursor/hooks/aftertone-install-dir` after global install). Set by hooks or env.
- **`SUPERTONIC_REPO`** — legacy alias (same path; older forks).

## Facts

- Assets: Hugging Face `Supertone/supertonic-3` via `fetch_assets.py` → `./assets/`.
- **Global install (default):** user hooks `~/.cursor/hooks.json` → wrapper → `~/aftertone/.cursor/hooks/speak_summary.sh`. Config TOML + daemon state stay under **`~/aftertone/.cursor/hooks/`**. Slash commands copied to `~/.cursor/commands/`.
- **Legacy:** project `.cursor/hooks.json` via `install.sh --into` (duplicates `py/` per repo).

## Cursor spoken summaries (`afterAgentResponse` + `tts_daemon`)

- **Reference:** [.cursor/hooks/README.md](.cursor/hooks/README.md) — every `speak_summary.toml` key, valid `lang` codes, heuristics, `quiet_hours`, **start / stop / status / restart**, when to restart the daemon.
- **Flow:** Cursor **`afterAgentResponse`** runs [.cursor/hooks/speak_summary.sh](.cursor/hooks/speak_summary.sh) → [py/speak_summary_prepare.py](py/speak_summary_prepare.py) (inline `text` from the hook; `stop` often has no useful transcript) → `POST` [py/tts_daemon.py](py/tts_daemon.py). Models stay loaded in the daemon.
- **Config:** [.cursor/hooks/speak_summary.toml](.cursor/hooks/speak_summary.toml) — **`voice_type`** / **`voice_style`**, **`lang`**, **`speed`**, **`total_step`**, `use_gpu`, `quiet_hours`, `min_chars`, **`max_chars`**, **`spoken_summary_max_chars`**, **`heuristic_max_chars`**, **`plain_excerpt_max_chars`**, heuristic keys, **`only_speak_spoken_summary`**, `mode`, `enabled`.
- **Control:** `cd py && uv run python tts_daemon_ctl.py {start|stop|status|restart} --repo-root ..` — PID/port under `.cursor/hooks/state/`.
- **Toml vs running daemon:** **`port`**, **`onnx_dir`**, **`voice_*`**, **`use_gpu`** need **`restart`**. The hook posts to **`state/tts-daemon.port`**; mismatch with TOML logs **`port_mismatch`** in `speak_summary-hook.log`. **`status`** prints TOML on disk + healthz.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omarelkhal/aftertone](https://github.com/omarelkhal/aftertone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
