---
trigger: always_on
description: This repository is small, script-driven, and intentionally low-dependency.
---

This repository is small, script-driven, and intentionally low-dependency.
Agentic coding work should stay conservative: preserve existing behavior, keep changes focused, and prefer direct reasoning over framework-heavy rewrites.

## Repository Overview

- `widevine_patch.py`: standalone Python 3 ELF patcher for `libwidevinecdm.so`.
- `install.sh`: POSIX shell installer for Chromium, Widevine extraction, and system integration.
- `uninstall.sh`: inverse of the installer; removes files and restores package state.
- `tests/unit/`: pytest-based unit tests for the Python patcher.
- `tests/unit/fixtures/minimal_elf.py`: synthetic ELF fixture generator used by unit tests.
- `tests/integration/test_installer.bats`: Bats integration tests for the installer.
- `tests/e2e/test_playback.py`: Playwright-based smoke tests for real hardware/browser playback.

## Rules Sources

- No `.cursorrules` file exists in the repository.
- No `.cursor/rules/` directory exists in the repository.
- No `.github/copilot-instructions.md` file exists in the repository.
- Therefore, this file is the primary agent instruction source inside the repo.

## Project Shape

- Language/runtime: Python 3 and POSIX `sh`.
- Test stack: `pytest`, `bats`, and `playwright`.
- Build system: none; package manager metadata is absent.
- Prefer minimal patches and preserve the current single-file utility architecture.
- Avoid new dependencies unless clearly necessary.
- Treat `install.sh` and `uninstall.sh` as system-touching code: explicit, reversible, careful.

## Build, Run, And Test Commands

There is no compile/build step in the conventional sense. The main workflows are direct script execution and tests.

### Primary Script Execution

- Run patcher help path: `python3 widevine_patch.py`
- Run patcher on a binary: `python3 widevine_patch.py input.so output.so`
- Run patcher with debug logging: `python3 widevine_patch.py --debug input.so output.so`
- Run installer: `sudo ./install.sh`
- Run installer in verbose mode: `sudo ./install.sh -v`
- Run uninstaller: `sudo ./uninstall.sh`
- Run uninstaller in verbose mode: `sudo ./uninstall.sh -v`

### Unit Tests

- Run all unit tests: `pytest tests/unit/ -q`
- Run one test file: `pytest tests/unit/test_widevine_patch.py -q`
- Run one test class: `pytest tests/unit/test_widevine_patch.py::TestAtomicRelocs -q`
- Run one specific test: `pytest tests/unit/test_widevine_patch.py::TestAtomicRelocs::test_both_atomic_relocs_rewritten -q`
- Run tests matching a substring: `pytest tests/unit/test_widevine_patch.py -k relr -q`
- Show stdout while debugging: `pytest tests/unit/test_widevine_patch.py -q -s`

### Integration Tests

- Run installer integration suite: `bats tests/integration/test_installer.bats`
- Run a filtered Bats test: `bats tests/integration/test_installer.bats --filter 'fails with non-zero exit on non-aarch64 architecture'`
- Useful when iterating on installer behavior: run the full Bats file after any `install.sh` change.

### End-To-End Tests

- Install Playwright first if needed: `pip install playwright`
- Install browser support: `playwright install chromium`
- Run all E2E tests: `pytest tests/e2e/ -q`
- Run headed E2E tests: `pytest tests/e2e/ -q --headed`
- Run one E2E test: `pytest tests/e2e/test_playback.py::test_spotify_loads_without_drm_error -q -s`

### Suggested Validation Matrix

- If you change `widevine_patch.py`, run at least `pytest tests/unit/test_widevine_patch.py -q`.
- If you change `install.sh`, run `bats tests/integration/test_installer.bats`.
- If you change browser-installation, playback, or user-facing verification guidance, consider `pytest tests/e2e/ -q` on appropriate hardware.

## Formatting And Linting

- No formal linter or formatter is configured in the repo.
- Do not invent a repo-wide formatting sweep.
- Match the surrounding file's style instead of enforcing a new one.

## Python Style Guidelines

- Target plain Python 3 without external dependencies.
- Keep `widevine_patch.py` self-contained.
- Prefer top-level helper functions over classes unless state management becomes unavoidable.
- Use descriptive snake_case names for functions and variables.
- Use ALL_CAPS for ELF constants, offsets, and other protocol-level constants.
- Keep low-level binary helpers short and obvious.
- Prefer explicit integer math and offsets over clever abstractions.
- Favor readability over DRY when manipulating binary layouts; duplicated clarity is acceptable here.
- Preserve existing script-style CLI flow in `widevine_patch.py` unless the user asks for modularization.

## Python Imports

- Group imports as: standard library, third-party, then local test-only imports.
- Keep imports simple and static when possible.
- In tests, the existing `sys.path.insert(...)` fixture import pattern is acceptable because the fixture is not packaged.
- Avoid adding optional imports to production code unless they are guarded and necessary.

## Python Formatting Conventions

- Follow the local file's quote style; do not normalize unrelated strings.
- Keep lines readable; wrap long assertions/messages with parentheses or continued expressions.
- Use blank lines to separate phases and helper sections, as the current files do.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xesco/pivine](https://github.com/xesco/pivine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
