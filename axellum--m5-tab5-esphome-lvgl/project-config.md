---
trigger: always_on
description: This file is the entry point for any AI agent (Claude, Codex, Cursor, Copilot, Gemini CLI, or a human skimming fast) working on this repository. Keep it short; when it drifts from the code, the code wins — fix this file, don't work around it.
---

# AGENTS.md — instructions for AI coding agents

This file is the entry point for any AI agent (Claude, Codex, Cursor, Copilot, Gemini CLI, or a human skimming fast) working on this repository. Keep it short; when it drifts from the code, the code wins — fix this file, don't work around it.

## What this project is

A Home Assistant dashboard running natively as ESP32-P4 firmware (ESPHome + LVGL 9.5 — not 8.x: check `lv_version.h` in the build tree before picking an API) on a M5Stack Tab5 V2. Push-only: the device never polls, Home Assistant automations push data via ESPHome API service calls. No web stack, no browser. See [`README.md`](README.md) for the full picture.

## Read this before touching anything

In order, before editing code or answering questions about architecture:

1. [`CARTOGRAPHIE_TAB5.md`](CARTOGRAPHIE_TAB5.md) — full dependency graph, file-by-file inventory, and a verified list of known technical debt / dead code. Read this first instead of reverse-engineering the YAML tree from scratch.
2. [`Tab5/README.md`](Tab5/README.md) — file-by-file description of the ESPHome packages, the HA service contract table, the globals table, and **8 mandatory code rules**. The eight game consoles are documented separately in [`docs/arcade.md`](docs/arcade.md) — read it only when touching a game.
3. [`docs/decisions/`](docs/decisions/README.md) — why non-obvious architectural choices were made (push-only, single-page UI, no hardcoded colors, etc.). Check here before "fixing" something that looks wrong.
4. [`docs/troubleshooting.md`](docs/troubleshooting.md) — incidents already diagnosed on this exact device. Check here before re-diagnosing a symptom that looks familiar (black screen after reboot, missing weather/planning data, mic pipeline stuck, etc.).
5. The target file itself, including its `[AI-CONTEXT]` header (see below).

## The `[AI-CONTEXT]` / `[AI-WARNING]` / `[AI-DEBUG]` convention

Most files in `Tab5/` and every file in `Tab5/ui_components/` open with an `[AI-CONTEXT]` comment block: role, architectural constraints, and `@ai_instruction` notes for common edits.

- **`[AI-WARNING]`** marks code that looks like a bug or anti-pattern but is a deliberate, tested fix (e.g. the blocking `delay(1000)` in `on_boot` for the display reset GPIO expander, or the non-wrapping forecast pagination). **Read the warning before "fixing" it.** At least one of these was already reverted once after an LLM audit "corrected" it without reading the comment.
- **`[AI-DEBUG]`** marks a good observation point for diagnosing issues at runtime (a log line, a diagnostic entity, a technique that already worked). See [`docs/debugging.md`](docs/debugging.md).

If you introduce a new non-obvious constraint, add or extend the relevant `[AI-CONTEXT]` block — don't leave the reasoning only in a commit message.

## Build & verify

```bash
# Compile — must succeed before any commit. Toolchain is cached locally (~20-45s).
python -m esphome compile tab5-ha-hmi.yaml
```

- If you modified a file included via `!include` (anything in `Tab5/ui_components/`), run `esphome clean` before the next `esphome run` — stale build cache is a known ESPHome trap.
- Compare the reported `config_hash` before/after a refactor that should be behavior-neutral — identical hash is the standard proof of "no functional change" used across this project's PR history.
- OTA-flashing the real device is a deliberate, human-authorized action, not a default step of a coding task — only do it if explicitly asked. If you do: confirm afterward via the device's own diagnostic entities (`ha_api_status`, boot time on `Tab5 Uptime` unchanged after the flash's own reboot, no reboot) rather than assuming success.
- `esphome compile` (schema + C++ compile) is the correctness gate for the firmware itself. CI (`.github/workflows/esphome-tab5.yml`) runs on every PR and on pushes to `main`: a `python` job (pre-commit hooks, pytest, the real Go engine built with g++, demo dry-run) and the same compile with a throwaway signing key (`openssl genrsa`, no `secrets.yaml` since lot 6b) **only when `tab5-ha-hmi.yaml`, `Tab5/` (Markdown excluded) or the workflow change**. `python`, `build` and `build-min` are required checks; `build` stays present and reports "skipped" (= success) on docs-only changes; `workflow_dispatch` forces a compile. The compile deliberately uses ESPHome `latest` (free upstream canary, [ADR-0016](docs/decisions/0016-ci-esphome-latest-canary.md)) and keeps its ccache between runs; a `build-min` job compiles the same configuration with the ESPHome version read from `min_version:` in `tab5-ha-hmi.yaml` (required too since 2026-09-26). `main` publishes the `tab5-firmware` artifact. A non-required job, `sanitizers.yml`, compiles the virtual tablet (`tab5-rendu-host.yaml`) with ASan + UBSan and fuzzes the 20 services, replays the out-of-range cases with the popups open and opens every screen; it fails on the first report read from the tablet's own output (`tools/sanitizers/`), after a positive control — read it before merging a C++ change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Axellum/M5-Tab5-ESPHome-LVGL](https://github.com/Axellum/M5-Tab5-ESPHome-LVGL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
