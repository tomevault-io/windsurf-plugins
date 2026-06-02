---
trigger: always_on
description: > This file is the authoritative guide for AI coding agents working on the **Kimiko** repository.
---

# AGENTS.md — Kimiko Project Operational Guide

> This file is the authoritative guide for AI coding agents working on the **Kimiko** repository.
> Kimiko is a cross-platform configuration-management repository that packages the reproducible parts of a `~/.kimi` directory for the [Kimi Code CLI](https://www.moonshot.cn/) by MoonshotAI.
> It is **not** a traditional application codebase. There is no `pyproject.toml`, `package.json`, or `Cargo.toml` at the root.
> The only buildable code is the standalone Python CLI under `validator/`.

---

## Project Overview

| Property | Value |
|---|---|
| **Name** | Kimiko |
| **Purpose** | Cross-platform installer and validator for the Kimi Code CLI `~/.kimi` mandate configuration |
| **Platforms** | macOS, Linux, WSL, Git Bash (Windows), PowerShell (Windows) |
| **CLI Version** | 1.46.0 (cached in `config/latest_version.txt`) |
| **Runtime Target** | Python 3.13 package `kimi-cli` (installed via `uv` into `~/.local/share/uv/tools/kimi-cli/`) |
| **Primary Languages** | TOML, YAML, Bash, PowerShell, Python, JSON |
| **License** | See `LICENSE` at repository root |

This repository stores:
- Runtime configuration (`config.toml`, `kimi.toml`)
- System agent mandate specifications (`mandate-agent.yaml`, `mandate-kimiko-agent.yaml`)
- Shell integration scripts (`.sh` and `.ps1` pairs)
- A configuration validator tool (`validator/`)
- Documentation (`docs/`)

The installed `~/.kimi` directory also holds OAuth credentials, session data, logs, and telemetry created by the Kimi CLI at runtime. These are **not** committed to the repository.

---

## Technology Stack

| Layer | Technology | Notes |
|---|---|---|
| **Application Runtime** | Python 3.13 `kimi-cli` | Installed via `uv`; source lives in site-packages, not this repo |
| **Config Formats** | TOML (primary), YAML (agent specs), JSON (schemas, registry) | |
| **Build Tool** | GNU/BSD `make` | Root `Makefile` handles cross-platform install/validation |
| **Validator** | Python 3.11+ | Uses built-in `tomllib`; `jsonschema`, `pyyaml`, `pytest`, `ruff` |
| **Schema Standard** | JSON Schema Draft 2020-12 | 6 schema files under `validator/schemas/` |
| **CI/CD** | GitHub Actions | `.github/workflows/ci.yml` |
| **Pre-commit** | `pre-commit` framework | `.pre-commit-config.yaml` with `ruff` and generic hooks |
| **Dependency Updates** | Dependabot v2 | Configured in `.github/dependabot.yml` |

---

## Directory Layout

### Repository source layout

```
kimiko/
├── Makefile                    # Root cross-platform installer & validator driver
├── README.md                   # Human-facing project overview
├── LICENSE                     # License file
├── .gitignore                  # Git ignore rules
├── .gitattributes              # Git attributes
├── .pre-commit-config.yaml     # Pre-commit hooks configuration
├── config/                     # Maps to ~/.kimi root when installed
│   ├── config.toml             # Primary runtime configuration (~1,483 lines)
│   ├── kimi.toml               # Hardened mirror of config.toml
│   ├── kimi.json.template      # Template for work-directory registry
│   ├── latest_version.txt      # Cached CLI version string ("1.46.0")
│   ├── mandate-agent.yaml      # System agent spec
│   └── mandate-kimiko-agent.yaml  # Hardened mirror of mandate-agent.yaml
├── scripts/                    # Maps to ~/.kimi root when installed
│   ├── activate-mandate.sh     # Mandate env var exporter + verifier
│   ├── activate-mandate.ps1    # PowerShell equivalent
│   ├── kimi-wrapper.sh         # KIMI binary wrapper (--yolo enforcement)
│   ├── kimi-wrapper.ps1        # PowerShell equivalent
│   ├── kimi-shell-integration.sh   # Shell profile integration
│   ├── kimi-shell-integration.ps1  # PowerShell equivalent
│   ├── launch-with-mandate.sh  # Banner launcher
│   ├── launch-with-mandate.ps1 # PowerShell equivalent
│   ├── INSTALL-GITBASH.md      # Git Bash-specific install guide
│   └── INSTALL-WSL.md          # WSL-specific install guide
├── docs/                       # Documentation
│   ├── AGENTS.md               # This file (installed to ~/.kimi/AGENTS.md)
│   ├── ARCHITECTURE.md         # Architecture Decision Records (ADRs)
│   ├── CHANGELOG.md            # Keep-a-Changelog format
│   ├── CONTRIBUTING.md         # Contribution guidelines
│   ├── CODE_OF_CONDUCT.md      # Contributor Covenant v2.1
│   ├── INSTALL-WINDOWS.md      # Windows installation guide
│   ├── RUP.md                  # Repository Upgrade Protocol
│   ├── SECURITY.md             # Security policy & reporting
│   ├── TODO.md                 # Bug hunt tracker
│   ├── TROUBLESHOOTING.md      # Platform-specific troubleshooting
│   └── legal/
│       └── DISCLAIMER.md       # Binding liability waiver
├── validator/                  # Only buildable code project
│   ├── Makefile                # Validator subproject automation
│   ├── README.md               # Validator documentation
│   ├── requirements.txt        # Python dependencies
│   ├── validate_kimi.py        # Main CLI entry point (~633 lines)
│   ├── schemas/                # JSON Schema files (Draft 2020-12)
│   │   ├── config-schema.json
│   │   ├── config-zero-blocker-schema.json
│   │   ├── credentials-schema.json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spearchucker667/kimiko](https://github.com/spearchucker667/kimiko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
