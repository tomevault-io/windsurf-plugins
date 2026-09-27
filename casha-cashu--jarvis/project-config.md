---
trigger: always_on
description: Russian voice assistant for Linux/macOS. DE/WM adapters (`i3`/`sway`/`hyprland`/`kde`/`gnome`/`macos`),
---

# JARVIS — agent guide

Russian voice assistant for Linux/macOS. DE/WM adapters (`i3`/`sway`/`hyprland`/`kde`/`gnome`/`macos`),
STT (Vosk / faster-whisper) + Silero VAD, commands-or-LLM routing (Ollama / OpenAI / Anthropic / OpenRouter), TTS (Piper / gTTS / SpeechT5).

## Layout

- `jarvis/` — Python backend (`cli.py` entry, `jarvis.cli:main`). `Jarvis` in `jarvis/__init__.py` is a thin orchestrator only.
- `jarvis/modules/` — `config_loader`, `audio_pipeline` (STT/VAD), `response_pipeline` (commands→LLM→TTS), `conversation_manager` (wake/mute/multi-turn), `lifecycle`, `nlu`, `bash_agent`, `commands`, `llm`, `reminder`, `dictation`, STT/TTS/VAD.
- `jarvis/adapters/` — one per platform. `jarvis/ui_bridge.py` — stdin/stdout JSON bridge spawned by the Tauri UI.
- `tests/` (+ `tests/integration/`), `data/commands.json` + `data/apps.json` (NLU training data), `docker/`, `jarvis-ui/` (Tauri 2 + React 19), `dist/arch/PKGBUILD`.
- `venv/` is the shared venv. `config.yaml` + `.env` are personal and gitignored (templates: `config.example.yaml`, `.env.example`). `HANDOFF.md` is gitignored session state — read it for in-progress context, never commit it.

## Commands (repo root)

- Unit tests: `PYTHONPATH=. ./venv/bin/python -m pytest -m "not slow and not integration" -q` (`make test` is the same with system `python3`; prefer the venv binary).
- Single test: `PYTHONPATH=. ./venv/bin/python -m pytest tests/test_env.py -v`.
- **Never run bare `pytest tests/`** — `tests/integration/` opens real audio/display devices and hangs. macOS CI uses the same `-m "not slow and not integration"` filter.
- Gate before done: `./venv/bin/ruff check jarvis/ tests/ && ./venv/bin/ruff format --check jarvis/ tests/ && ./venv/bin/python -m mypy jarvis/ && cargo check --manifest-path jarvis-ui/src-tauri/Cargo.toml` (mypy scope is `jarvis/` only — pre-commit excludes `tests/`/`venv/`/`build/`).
- Frontend: `cd jarvis-ui && npm run build` (tsc+vite) + `npm run lint` (oxlint) + `npm run test` (vitest).
- Docker unit tests (install everything; what CI runs): `make docker-test-arch` / `docker-test-debian` / `docker-test-fedora`.
- Docker integration: `make docker-integration-i3` (runs with `--cap-add=SYS_PTRACE --cap-add=SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined`) / `make docker-integration-sway` (needs `--privileged`).
- Run: `source venv/bin/activate && jarvis run`; `jarvis run --dry-run` skips STT/TTS/VAD model loads. `jarvis doctor` dumps env/config/audio/models/LLM state — ask for its output when debugging user reports. GUI dev: `cd jarvis-ui && npm run tauri dev`.

## Architecture

- `config_loader.py` — YAML load + `${VAR}` expansion (missing var → warning + empty string) + pydantic validation (`config_schema.py`).
- `audio_pipeline.py` — STT/VAD lifecycle; `dry_run=True` skips model loads.
- `response_pipeline.py` — commands → LLM → TTS routing. `conversation_manager.py` — wake word, mute, multi-turn. `lifecycle.py` — SIGINT/SIGTERM + ordered shutdown.
- `modules/nlu.py` — `IntentRouter.parse()` → `{raw, intent, intent_confidence, slots}`; TF-IDF + LogisticRegression trained at startup from `data/*.json`, cacheable via `JARVIS_NLU_CACHE` (cache dir `0700`, files `0600`). Old fuzzy/pattern path in `commands.py` is fallback only.
- `modules/bash_agent.py` — LLM automation with 3 layers: hardline blocklist → dangerous-pattern detector → approval gate (`auto`/`strict`/`yolo`). Tools `bash`/`read`/`write` (write blocks `/etc` `/usr` `/boot` `/sys` `/proc` `/dev`).
- `modules/llm.py` — all providers share one history file (`HISTORY_FILE`, default `~/.local/share/jarvis/history.json`, override `JARVIS_HISTORY_FILE`), clamped to `llm.max_history`; `clear_history()` is atomic temp+rename. `LLMClient` is `ABC` with abstract `chat()` — never instantiate raw.
- `modules/commands.py` — `CommandExecutor._run`: `cmd` may be a string or a **callable returning str** (evaluated at execute time); blocks for `commands.execution_timeout` (default 30s), then SIGTERM → SIGKILL after 2s grace.

## Hard rules

- **No `shell=True`** in `jarvis/`. Adapter command strings go through `shlex.split` + `subprocess.*(env=sanitized_env())`.
- **Every `subprocess.*` passes `env=sanitized_env()`** (`jarvis/_env.py` allowlist). API keys flow as kwargs via `config.yaml` → `provider_config` → `LLMManager`; never put them in `os.environ` (`cli_helpers` must not write there either).
- **CI: never swallow the tested command's exit code** (`|| true`, `set +e`, pipes without `pipefail`, skip-on-empty are for cleanup/best-effort daemons only). Any binary used by tests/entrypoints must be installed in that image. macOS `test.yml` needs `set -o pipefail` before `pytest … | tail`.
- **Docker: manifests before sources** — `COPY pyproject/requirements` + install first, then `COPY . .`; torch always CPU (`--index-url …/whl/cpu`), PyPI default pulls a CUDA bundle.
- Never commit secrets or session state: `.env`, `config.yaml`, `SESSION.md`, `HANDOFF.md` are gitignored — keep them that way.

## Python / test gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [casha-cashu/jarvis](https://github.com/casha-cashu/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
