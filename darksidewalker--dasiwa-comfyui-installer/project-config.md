---
trigger: always_on
description: This document is the authoritative reference for any AI agent working on this codebase.
---

# 🤖 AGENTS.md — DaSiWa ComfyUI Installer: Agent Operational Protocol

This document is the authoritative reference for any AI agent working on this codebase.
Read it in full before making any changes. It defines the architecture, public APIs,
invariants that must never be broken, and the correct modification target for every
category of change.

---

## 1. Project Philosophy

The installer operates on three core invariants that every agent must preserve:

1. **Zero system pollution.** Nothing is installed into the user's system Python or global
   PATH. All packages live inside `ComfyUI/venv/`. All package management goes through `uv`,
   never `pip` directly.

2. **Ask once, never again.** The pre-flight wizard (`preflight_wizard` in `setup_logic.py`)
   collects every interactive decision up front and produces a `plan` dict. After the wizard
   confirms, the installation runs unattended. Do not add `input()` calls anywhere else in
   the install flow.

3. **Config files are never mutated at runtime.** `config.json` is the committed global
   default. `config.local.json` is the user override layer. Neither file is written to during
   an install run. In-memory overrides (e.g. the CUDA downgrade for SageAttention on Windows)
   are applied to derived variables only.

---

## 2. Repository Layout

```
dasiwa-comfyui-installer/
├── install.ps1              # Windows bootstrapper (uv + Python acquisition)
├── install.sh               # Linux bootstrapper
├── install_comfyui.py       # Self-update sentinel (hash guard + execv restart)
├── setup_logic.py           # Main orchestrator — wizard → plan → execution
├── config.json              # Global defaults (COMMITTED, never written at runtime)
├── config.local.json        # User overrides (GITIGNORED, deep-merged over config.json)
├── custom_nodes.txt         # Default node list (COMMITTED)
├── custom_nodes.local.txt   # User node override (GITIGNORED, takes full precedence)
└── utils/
    ├── logger.py            # UI layer — all terminal output and prompts
    ├── hardware.py          # GPU detection and vendor classification
    ├── comfyui_clone.py     # ComfyUI git management
    ├── downloader.py        # File downloads and model migration
    ├── task_nodes.py        # Custom node cloning and dependency install
    ├── task_ffmpeg.py       # FFmpeg acquisition (portable Win / apt Linux)
    ├── task_sageattention.py# SageAttention install (wheel-first, source fallback)
    └── reporter.py          # Final installation summary
```

---

## 3. Module Catalogue & Public API

### 3.1 `utils/logger.py` — **Chronicler**

The single source of truth for all terminal output and user prompts. No module may call
`print()` or `input()` directly except `logger.py` itself.

| Method | Signature | Purpose |
| :----- | :-------- | :------ |
| `Logger.init()` | `()` | Must be called once at startup. Enables Windows VT sequences; respects `NO_COLOR`. |
| `Logger.log()` | `(text, level, bold)` | Core output. Levels: `info` `ok` `warn` `fail` `error` `done` `magenta` `debug`. |
| `Logger.error()` | `(text)` | Bold red `[-]` prefix. |
| `Logger.warn()` | `(text)` | Yellow `[!]` prefix. |
| `Logger.success()` | `(text)` | Green `[DONE]` prefix. |
| `Logger.info()` | `(text)` | Cyan `[*]` prefix. |
| `Logger.debug()` | `(text)` | Gray `[.]` prefix. Low-signal detail only. |
| `Logger.banner()` | `(title, subtitle, width)` | Double-box header for major phase breaks. |
| `Logger.section()` | `(title, width)` | Single-rule header for sub-phases. |
| `Logger.rule()` | `(width)` | Horizontal divider. |
| `Logger.kv()` | `(key, value, width)` | Aligned key/value pair for summaries. |
| `Logger.ask()` | `(question, default)` | Plain text prompt with optional default. |
| `Logger.ask_yes_no()` | `(question, default)` | Boolean Y/N prompt. Returns `bool`. |
| `Logger.ask_choice()` | `(question, options, default_index)` | Numbered menu. `options` = list of `str` or `(label, description)`. Returns chosen index `int`. |
| `Logger.spinner()` | `(message)` | Context manager. Shows animated spinner while body runs; falls back to plain log when not a TTY. |

**Agent rules:**
- Always call `Logger.init()` before any output.
- Use `Logger.ask_yes_no` / `Logger.ask_choice` for all prompts. Never raw `input()`.
- `Logger.spinner()` is for operations that produce no stdout (e.g. hashing, probing). Use
  `run_cmd(..., stream=True)` for long subprocesses where the user needs to see progress.

---

### 3.2 `setup_logic.py` — **Orchestrator**

The entry point for the actual installation. Contains the wizard, the execution engine,
and all high-level task orchestration.

#### Key functions

| Function | Purpose |
| :------- | :------ |
| `main()` | Entry point. Loads config → pre-flight → wizard → executes plan. |
| `preflight_wizard(config_data, current_dir, hw)` | Collects all user decisions. Returns the `plan` dict (see §3.2.1). |
| `_detect_existing_state(current_dir)` | Non-destructive probe of disk. Returns `state` dict describing what is already installed. |
| `_resolve_cuda_target(config_data, want_sage)` | Applies the Windows/SageAttention/CUDA-13 downgrade rule. Returns `(cuda_version_str, was_downgraded_bool)`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darksidewalker/dasiwa-comfyui-installer](https://github.com/darksidewalker/dasiwa-comfyui-installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
