---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

SoMatic is an agent-first CLI for native desktop UI automation using Set-of-Marks (SoM) screenshot technology. It exposes a JSON-only command interface (`somatic <command>`) that AI agents use to control native desktop UIs — clicking, typing, scrolling, and capturing annotated screenshots where UI elements are numbered for reference.

Mark resolution is purely id → coordinate. Marks carry no captions or OCR text; agents act on numbered boxes and the click path looks up the center of the chosen mark from a cached session JSON.

## Commands

```bash
# Install Python package (run once)
pip install -e .[dev,vision,mcp]

# Run all tests
python -m pytest

# Run a single test file
python -m pytest tests/test_yolo_onnx_provider.py

# Run a single test by name
python -m pytest tests/test_cli.py::test_wait_outputs_json -q

# npm postinstall / shim validation
node bin/somatic.js doctor
npm run pack:check
```

The `[vision]` extra installs `onnxruntime`, `numpy`, `huggingface-hub`, plus `ultralytics`+`torch` (used only by the first-run `.pt → .onnx` conversion fallback).

## Architecture

SoMatic has **three agent-facing surfaces** that share the same underlying modules, plus two long-lived background processes that they orchestrate:

1. **Plain CLI** (`somatic <command>`) — universal fallback for any harness that can shell out. JSON to stdout; for screenshots the JSON now includes `image_b64`/`annotated_image_b64` so the bytes can be fed to the model without a separate Read step.
2. **MCP server** (`somatic mcp serve` or `python -m somatic.mcp_server`) — for Claude Code / Cursor / Continue. Tools mirror the CLI verbs; screenshot tools emit MCP `ImageContent` + `TextContent` so the agent sees the image inline. Ships a `skill` MCP prompt that loads the operating loop.
3. **Vision daemon** (`vision_init` / `vision_stop`) — long-lived background process that holds the YOLO ONNX model. Both the CLI and MCP server talk to it over HTTP on `127.0.0.1:8765`.
4. **Headless session** (`headless start` / `headless stop`, Linux only) — long-lived Xvfb + WM + optional VNC + optional apps. When active, every other CLI invocation transparently inherits `DISPLAY` / `XAUTHORITY` / `DBUS_SESSION_BUS_ADDRESS` via `headless.apply_active_env()` at the top of `main()`. The vision daemon is auto-restarted under the headless display so screenshots come from the virtual desktop.

A thin Node.js shim (`bin/somatic.js`) spawns the Python CLI for npm-installed users; it's a pure passthrough.

```
bin/somatic.js          Node shim — resolves Python, sets PYTHONPATH=src/, spawns
scripts/postinstall.js  npm postinstall: creates .venv, pip installs .[vision]
src/somatic/            MIT-licensed runtime. ZERO AGPL imports (CI-enforced).
  cli.py                Argparse dispatcher — calls headless.apply_active_env() first
  mcp_server.py         FastMCP wrapper exposing the same surface as MCP tools+prompt
  skill.py              Single source of truth for SKILL.md content (used by CLI+MCP)
  licenses.py           Static MIT/AGPL notices used by `somatic license` + MCP prompt
  SKILL.md              Packaged operating-loop guidance (shipped in the wheel)
  automation.py         PyAutoGUI wrappers (click, click_near, type, move, drag, scroll, hotkey…)
  screenshot.py         Capture + annotation; embeds base64 PNG bytes in response
  marks.py              Mark dataclass, session cache (JSON file), normalize_marks()
  vision_client.py      HTTP client for the local vision daemon (GET /health, POST /parse)
  vision_daemon.py      Background process: loads ONNX model, serves on 127.0.0.1:8765
  providers/yolo_onnx.py  Pure inference; SHA256-verified HF download; no .pt→.onnx convert
  headless.py           Xvfb session lifecycle, state-file glue, env overlay
  doctor.py             Platform diagnostics (dependency checks, failsafe tests)
  paths.py              XDG-compliant runtime paths (cache, data, screenshots, PID files)
  jsonio.py             command_response(), SomaticError, fail() — all CLI output goes here
tools/                  AGPL-3.0-licensed boundary. NOT shipped in npm or PyPI.
  LICENSE.AGPL          Full AGPL-3.0 text
  README.md             License-boundary explanation
  requirements.txt      ultralytics + torch (AGPL-3.0 deps)
  convert_yolo_to_onnx.py  Maintainer/power-user .pt → .onnx conversion
tests/                  Including test_license_boundary.py which CI-enforces the MIT/AGPL split
```

### Licensing boundary

The repo follows the FFmpeg licensing strategy:

- The published artifacts (`npm install -g @somatic-cli/cli` and PyPI wheel/sdist) are pure MIT.
- The model weights are AGPL-3.0 (inherited from upstream YOLO). They are downloaded at runtime from a separately-licensed Hugging Face repository via `somatic vision init`; SoMatic never bundles them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Smyan1909/SoMatic](https://github.com/Smyan1909/SoMatic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
