---
trigger: always_on
description: This repository provides single-click installers for Langflow on Windows, macOS, and Linux using `uv` as the package manager. Python 3.12 is pinned. Langflow is pinned to version **1.11.3**.
---

# AGENTS.md — IBM Hacktiv8 / Langflow Installer for Windows, macOS, and Linux

## Project Overview

This repository provides single-click installers for Langflow on Windows, macOS, and Linux using `uv` as the package manager. Python 3.12 is pinned. Langflow is pinned to version **1.11.3**.

**Author**: Nikki Satmaka
- GitHub: https://github.com/NikkiSatmaka/
- LinkedIn: https://linkedin.com/in/nikkisatmaka/

## Repository Structure (root only)

| File | Purpose |
|------|---------|
| `AGENTS.md` | This file — agent guidance |
| `CONTRACT.md` | Formal requirements specification |
| `Install Langflow.bat` | Double-click launcher that bypasses execution policy (Windows) |
| `Install Langflow.command` | Double-click launcher (macOS) |
| `Install Langflow.sh` | Launcher (Linux) |
| `Stop Langflow.bat` | Double-click to stop the running Langflow server (Windows) |
| `Stop Langflow.command` | Double-click to stop the running Langflow server (macOS) |
| `Stop Langflow.sh` | Double-click to stop the running Langflow server (Linux) |
| `README.md` | This file — for humans |
| `CHANGELOG.md` | Release history |
| `src/install-langflow-script.ps1` | Main PowerShell installer/uninstaller script (Windows) |
| `src/install-langflow.sh` | Main bash installer/uninstaller script (macOS/Linux) |
| `src/stop-langflow-script.ps1` | PowerShell stop script (Windows) |
| `src/stop-langflow.sh` | Bash stop script (macOS/Linux) |
| `src/uv-install.ps1` | Fetched from astral.sh at package time (not in repo) — eliminates `irm \| iex` AV trigger (Windows only) |
| `src/stop-langflow-script.ps1` | PowerShell stop script (Windows) |
| `src/stop-langflow.sh` | Bash stop script (macOS/Linux) |
| `src/constraints.txt` | Pins known-breaking transitive deps that ship source-only releases without wheels; currently empty (all deps ship pre-built wheels on every target platform) |
| `mise.toml` | Dev tooling: pins shellcheck, shfmt, powershell and defines lint/fmt tasks |
| `.shellcheckrc` | shellcheck config (disables SC2059 for intentional ANSI colour output) |
| `PSScriptAnalyzerSettings.psd1` | PSScriptAnalyzer config (excludes rules that conflict with conventions) |
| `scripts/verify.sh` | Pre-commit verification checks (12 checks, POSIX-safe for CI) |
| `scripts/package.sh` | Cross-platform zip packaging (bash) |
| `scripts/package.ps1` | Cross-platform zip packaging (PowerShell) |
| `.github/workflows/verify.yml` | CI: PR verification (runs `scripts/verify.sh`) |
| `.github/workflows/install-test.yml` | CI: OS-matrix install test (win/mac/linux) — runs the real installer scripts end-to-end without build tools, validating binary wheels and the installer code paths before release; a shared reusable workflow (`workflow_call`) consumed by `release.yml` |
| `.github/workflows/release.yml` | CI: Automated release on tag push (verify + install test + package + publish) — calls `install-test.yml` as a reusable workflow |
| `docs/TROUBLESHOOTING.md` | Common issues and fixes |
| `docs/GATEKEEPER.md` | macOS Gatekeeper bypass guide |
| `docs/index.html` | Landing page for non-GitHub users (GitHub Pages) — published at `https://nikkisatmaka.github.io/langflow-installer-wrapper/` |

## Design Constraints

- **No admin rights required** — everything installs under `%USERPROFILE%`
- **Idempotent** — safe to re-run; checks before acting
- **User-prompted** — script asks Install / Uninstall / Quit at startup
- **Credits banner** — GitHub + LinkedIn displayed on every run (Chris Titus style)
- **Version pinned** — Langflow `==1.11.3`; do not change without updating CONTRACT.md
- **Cross-platform** — Windows (PowerShell), macOS, and Linux (bash); platform-specific logic with shared installer flow
- **Python pinned** — 3.12 via `uv python install 3.12` (only version with pre-built wheels for all C-extensions on Windows; 3.13+ requires MSVC not available to most users)

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| `uv` over `pip` | Faster, self-bootstrapping, no pre-installed Python needed |
| `%USERPROFILE%\.local\bin` added to permanent PATH | uv installer puts binaries there; script ensures it persists |
| `WScript.Shell` COM for shortcut | Standard Windows method, no external deps |
| Desktop shortcut targets `uv run langflow run` | Works regardless of active venv state |
| Uninstall keeps `uv` | uv may be used for other projects |
| UTF-8 BOM required on `.ps1` | Windows PowerShell requires UTF-8 with BOM; without it, non-ASCII characters cause parser errors |
| `uv-install.ps1` fetched at package time | Eliminates `irm \| iex` pattern that heuristic AV triggers on; uses `$PSScriptRoot` to reference local file |
| Release zip structure | `Install Langflow.bat` and `LICENSE` at zip root; `install-langflow-script.ps1`, `uv-install.ps1`, and `constraints.txt` under `src/` — mirrors repo layout |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NikkiSatmaka/langflow-installer-wrapper](https://github.com/NikkiSatmaka/langflow-installer-wrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
