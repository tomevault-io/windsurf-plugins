---
trigger: always_on
description: This file is a durable handoff for any agent working in this repo.
---

# AGENTS.md

This file is a durable handoff for any agent working in this repo.

- Always use `uv` for Python commands in this repo.
- Prefer updating both this file and CLAUDE.md when you learn something that a future agent would otherwise need the user to repeat.

## Maintaining This File

- If you discover a new “gotcha” (env var needed, a test that hangs, a required compile step, a script location), add it here.
- Keep notes command-first and loop-friendly (so the next agent can run/verify quickly).

## Agent Operational Notes

### Package manager / running

- Run app: `uv run aicodeprep-gui`
- Run tests: `uv run pytest`

### Licensing / Pro access

- Remote free-for-all Pro override: `https://wuu73.org/aicp/aicp-free-now.md` with `free=1` or `free=0`; cached for 15 minutes in `~/.aicodeprep-gui/settings.toml` under `pro_free_access`.
- The same remote Pro flag file also supports `message=` or `msg=` for a one-time startup popup and top banner announcement keyed to the exact message text.
- Verified paid licenses still short-circuit Pro gating even when the remote free flag is off or unreachable.
- Gumroad verification now uses `increment_uses_count=false` plus a 30s request timeout and longer retries so transient failures do not burn activations.

### AI endpoints

- Legacy `~/.aicodeprep-gui/ai-endpoints.toml` configs that point at `extra.wuu73.org/aimodels/v1` in either `http` or `https` form are benchmark/mock configs; that server returns canned non-stream and streaming responses.
- `aicodeprep_gui/pro/ai_assist/endpoint_config.py` now migrates that old default to an empty `Local / Custom Endpoint` so Chat and Flow Studio do not silently talk to the mock server.
- The shared compatible endpoint now lives in `~/.aicodeprep-gui/api-keys.toml` under `[custom]`; `endpoint_config.py` mirrors chat's `local` endpoint to/from that section so Flow Studio and AI Chat use the same URL/API key/model.
- If a user reports `This is a mock response from the fake LLM server` or `This is chunk 1/2/3`, inspect `~/.aicodeprep-gui/ai-endpoints.toml` first.

### GUI automation for agents (screenshots, loops, no manual closing)

This repo includes custom test infrastructure that allows agents to open the GUI, take screenshots, and reliably close windows during tests.

Key pieces:

- Environment variables used for tests:
  - `AICODEPREP_TEST_MODE=1` (set by pytest config)
  - `AICODEPREP_NO_METRICS=1` (set by pytest config)
  - `AICODEPREP_NO_UPDATES=1` (set by pytest config)
  - `AICODEPREP_AUTO_CLOSE=1` (optional; auto-closes test windows after ~10 seconds)

- Pytest configuration:
  - `tests/conftest.py` sets test-mode env vars early and handles Qt window cleanup between tests.

- Screenshot helpers:
  - `aicodeprep_gui/utils/screenshot_helper.py`
    - Screenshot output dir: `screenshots/test_captures/`
    - Useful functions: `capture_window_screenshot`, `capture_widget_screenshot`, `compare_screenshots`, `get_text_color_contrast`

- UI launch + screenshot harness:
  - `tests/test_helpers/screenshot_tester.py`
    - `ScreenshotTester.launch_and_capture()` opens the main window, processes events, optionally starts an auto-close timer, then captures a screenshot.

- Baseline screenshot test:
  - `uv run pytest tests/test_screenshot_baseline.py -v`

### Internationalization (i18n) and Accessibility (a11y)

**For all translation, language, and accessibility work, see [AGENTS_intl.md](AGENTS_intl.md)**

That file contains detailed workflows for:

- Adding/updating translations (`.ts` -> `.qm` compilation)
- Testing language switching with screenshots
- Adding new UI elements that need translation
- Troubleshooting translation issues
- Accessibility implementation guidance

Consult AGENTS_intl.md when you need to:

- Wrap new UI strings in `self.tr()`
- Complete translations for a language
- Test visual appearance across locales
- Work on keyboard navigation or screen reader support

## CLAUDE.md (Reference Copy)

The content below mirrors CLAUDE.md. If you update CLAUDE.md with new agent-relevant information, consider updating this section too.

---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Agent Quick Start (Read This First)

### Package Manager / Running Python

- Always use `uv` for Python commands in this repo.
- Examples:
  - Run app: `uv run aicodeprep-gui`
  - Run tests: `uv run pytest`
  - Run one test: `uv run pytest tests/test_i18n.py -q`

### Licensing / Pro access

- Remote free-for-all Pro override: `https://wuu73.org/aicp/aicp-free-now.md` with `free=1` or `free=0`; cached for 15 minutes in `~/.aicodeprep-gui/settings.toml` under `pro_free_access`.
- The same remote Pro flag file also supports `message=` or `msg=` for a one-time startup popup and top banner announcement keyed to the exact message text.
- Verified paid licenses still short-circuit Pro gating even when the remote free flag is off or unreachable.
- Gumroad verification now uses `increment_uses_count=false` plus a 30s request timeout and longer retries so transient failures do not burn activations.

### GUI Test Automation (No Manual Window Closing)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [detroittommy879/aicodeprep-gui](https://github.com/detroittommy879/aicodeprep-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
