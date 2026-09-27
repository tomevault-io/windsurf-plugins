---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Guiness is a desktop GUI-agent that drives real Android phones from a PC: user types a natural-language instruction, a VLM loops `screenshot → think → action` on the device until `Complete / Fail / max_steps`. Two entry points share the same engine:

- `main.py` — PySide6 desktop app (product entry). One conversation = one task.
- `run_eval.py` — JSONL batch CLI for regression/eval runs.

`ARCHITECTURE.md` has the detailed design notes (read it before non-trivial changes). `android/README.md` covers the companion Android APK ("Guiness Controller") used by WiFi mode.

## Commands

```bash
# Run from source (dev)
pip install -r requirements.txt
python main.py                           # GUI
python run_eval.py --config config.yaml  # CLI batch eval (reads task.task_file)
python run_eval.py --dry-run             # validate config + imports without running

# Package
bash build.sh                  # PC bundle (PyInstaller) + Android APK
bash build.sh --no-apk         # PC only
bash build.sh --no-pc          # APK only
bash build.sh --skip-deps      # skip pip install

# Android (companion APK, see android/README.md)
cd android && ./gradlew :app:assembleDebug
```

No test suite or linter is wired up. Validate changes by running `python main.py` (GUI) or `python run_eval.py --dry-run` (CLI imports/config sanity). For end-to-end: a real Android device + WiFi mode APK or USB ADB.

## Architecture (what to know before touching it)

Layers — lower does not depend on higher:

```
entry         main.py (GUI) │ run_eval.py (CLI)
factory       core/setup.py            build_components / build_runner / resolve_device_id
engine        runner/episode_runner.py EpisodeRunner
              reporter/                Reporter protocol (CLI colored output / GUI null)
capabilities  device/ action/ model/ apps/
infra         core/config.py  gui/paths.py  utils/
```

### Components (`core/setup.py`)

`build_components(device_id, device_type, model_config, mode, token, on_progress)` returns:

```python
@dataclass
class Components:
    backend: DeviceBackend     # usb → AdbBackend, wifi → WifiBackend
    inference: InferenceClient
    executor: ActionExecutor
```

> Note: `ARCHITECTURE.md` still describes an older `Components(adb, automator, inference, executor)` shape. The current code uses a single `backend` abstraction — trust the code.

`build_runner(components, config, output_dir, date_str, stop_check, on_step_complete, reporter)` wires those into an `EpisodeRunner`.

### DeviceBackend protocol (`device/backend.py`)

Upper layers (`ActionExecutor`, `EpisodeRunner`) depend on this Protocol only, never on `AdbBackend` / `WifiBackend` directly. Unsupported extensions raise `CapabilityUnsupported` so callers can degrade.

- **USB mode** → `device/adb_backend.py` (ADB + uiautomator2).
- **WiFi mode** → `device/wifi_backend.py` talks HTTP + WebSocket to the Android companion APK (`android/`), which uses AccessibilityService + MediaProjection on the phone. No ADB, no dev-options, no root.

### Episode lifecycle (`runner/episode_runner.py`)

`EpisodeRunner.run(task)` is three phases:

| Phase | Work |
| --- | --- |
| `_prepare_episode` | Fill metadata, create `output_dir/<date>/<app>/<episode_id>/`, init history |
| `_run_step` × N | screenshot → XML dump → compress → `inference.predict` → `normalize_action` → `executor.execute` → append `step_record` → `reporter.on_step_complete` + external callback |
| `_finalize_episode` | Deep-copy episode, strip `screenshot_time` (reporter-only field, does not hit disk), write `task.json`, clean tmp, issue `back_times` back presses |

Termination: `stop_check()` is True, `action.func ∈ {Complete, End, Speak, Fail}`, or `step >= max_steps`.

### Reporter (`reporter/base.py`)

```python
class Reporter(Protocol):
    def on_episode_start(self, task: dict) -> None: ...
    def on_step_complete(self, step_record: dict) -> None: ...
    def on_episode_finish(self, result: dict) -> None: ...
```

Engine never prints. CLI uses `reporter/cli_reporter.py` (ANSI). GUI uses `NullReporter()` and renders via a separate `on_step_complete` callback → Qt signal → `ChatFeed`.

### Config (`core/config.py`, schema in `config.yaml`)

Top-level keys: `device`, `operation`, `compress`, `model`, `task`. See `config.yaml` for the authoritative schema. `model.source` is `mify | custom` (each has its own prompt file — see below).

**Convention**: library code does not call `get_*_config()`. Entry points read the config once and pass sub-dicts explicitly (`compress` → `compress_image()`, `model` → `InferenceClient`). `get_config()` is for outermost entry only.

### Model inference (`model/`)

- `inference_client.py` branches on `source`:
  - `mify` → mify gateway + token auth, reads `model/prompts/mify.txt`
  - `custom` → any OpenAI-compatible endpoint, reads `model/prompts/custom.txt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeerRay-Lab/Xiaomi-GUI-0](https://github.com/SeerRay-Lab/Xiaomi-GUI-0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
