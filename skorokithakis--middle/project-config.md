---
trigger: always_on
description: - This file is for coding agents working in this repository.
---

# Middle agent guide

## purpose and scope
- This file is for coding agents working in this repository.
- Keep changes small, explicit, and consistent with existing code.
- This repository currently contains two code paths:
- `src/main.cpp`: ESP32-S3 firmware (Arduino via PlatformIO).
- `sync.py`: host-side BLE sync and transcription script (Python via `uv run --script`).

## repository facts discovered
- Build system: PlatformIO (`platformio.ini`).
- Firmware environment: `seeed_xiao_esp32s3`.
- Python runtime pattern: shebang with inline `uv` script metadata.
- No dedicated test suite is currently present.
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` files were found.

## cursor and copilot rules
- Cursor rules: none found in this repository.
- Copilot instructions: none found in this repository.
- If these files are added later, treat them as authoritative and update this guide.

## build, lint, and test commands
- Run all commands from repo root: `/home/stavros/Code/Hardware/middle`.

## firmware commands (platformio)
- Install PlatformIO Core if missing: `pipx install platformio` or equivalent.
- Build firmware: `pio run -e seeed_xiao_esp32s3`.
- Build and upload firmware over USB: `pio run -e seeed_xiao_esp32s3 -t upload`.
- Open serial monitor: `pio device monitor -b 115200`.
- Upload filesystem image (LittleFS): `pio run -e seeed_xiao_esp32s3 -t uploadfs`.
- Clean build artifacts: `pio run -e seeed_xiao_esp32s3 -t clean`.
- Print detected serial devices: `pio device list`.

## static analysis and linting
- There is no configured formatter or linter config checked into this repo.
- Use PlatformIO static checks for firmware when needed:
- `pio check -e seeed_xiao_esp32s3`.
- If you add a formatter or linter, document exact commands here.

## python script commands
- Run sync client (uses inline dependencies via uv): `uv run sync.py`.
- Optional dry import check: `uv run python -c "import sync"`.
- If you add tests later, keep `uv` as the default runner for consistency.

## test commands
- Current state: no automated tests are present.
- Firmware test harness command (if tests are added under PlatformIO):
- Run all firmware tests: `pio test -e seeed_xiao_esp32s3`.
- Run a single firmware test by name: `pio test -e seeed_xiao_esp32s3 -f <test_name>`.
- Python test harness command (if `pytest` is added):
- Run all python tests: `uv run pytest`.
- Run a single test file: `uv run pytest tests/test_sync.py`.
- Run a single test case: `uv run pytest tests/test_sync.py::test_case_name`.

## style guidelines

## source of truth
- Firmware behavior is defined by code in `src/main.cpp`, not by `ARCHITECTURE.md`.
- Host sync behavior is defined by code in `sync.py`, not by TODO notes.
- Update docs when behavior changes, but do not change behavior to match stale docs.

## general
- Prefer clear, direct code over abstractions.
- Keep functions focused and small when possible.
- Avoid speculative infrastructure and unused helpers.
- Keep behavior deterministic and explicit.
- Preserve current protocol and storage compatibility unless a task requires migration.

## comments and documentation
- Keep comments minimal and only for non-obvious intent.
- Prefer comments that explain why a choice was made.
- Remove comments that restate code line-by-line.
- When changing commands or workflows, update this file in the same change.

## naming
- Use descriptive names with domain meaning.
- Python: `snake_case` for functions and variables, `UPPER_SNAKE_CASE` for constants.
- C++ (this repo): keep existing local style (`snake_case` functions and variables, lower-case local class names currently used in `main.cpp`).
- BLE UUID constants should remain explicit and grouped.

## imports and includes
- Python imports belong at file top-level.
- Group Python imports as: standard library, third-party, local (if any).
- C++ includes belong at top of file.
- Do not add lazy imports inside functions unless there is a measured startup reason.

## typing and interfaces
- Python: add type annotations for public functions and new helpers.
- Preserve existing typed return signatures such as `tuple[int, list[Path]]`.
- Prefer concrete built-in collection types (`list`, `dict`, `tuple`).
- C++: use explicit fixed-width types where protocol size matters (`uint8_t`, `uint16_t`, `uint32_t`).

## formatting
- Match existing formatting in each file.
- Python: follow PEP 8 style and keep line lengths readable.
- C++: keep brace and spacing style consistent with `src/main.cpp`.
- Keep logs concise and prefixed by subsystem tags already in use (for example `[ble]`, `[rec]`, `[flash]`).
- Serial output must use `\r\n` line endings (e.g. `Serial.printf("...\r\n")` and `Serial.println()` — note that `println` already appends `\r\n`).

## error handling
- Do not silently swallow errors.
- Fail fast for invariant violations.
- For device and BLE operations, log actionable context before returning.
- Keep recovery paths simple and explicit.
- Avoid broad exception handling unless boundary code needs it.

## firmware-specific conventions
- Maintain current sample rate and file format unless intentionally changing protocol.
- Recording storage format is raw unsigned PCM8 and is consumed by `sync.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skorokithakis/middle](https://github.com/skorokithakis/middle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
