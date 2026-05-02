---
trigger: always_on
description: **Decode-Orc** is a cross-platform orchestration framework for LaserDisc and tape decoding workflows. It provides both a GUI (`orc-gui`) and CLI (`orc-cli`) interface, sharing a common MVP-architected core (`orc/core`, `orc/presenters`, `orc/view-types`).
---

# Copilot Instructions for Decode-Orc

## Repository Overview

**Decode-Orc** is a cross-platform orchestration framework for LaserDisc and tape decoding workflows. It provides both a GUI (`orc-gui`) and CLI (`orc-cli`) interface, sharing a common MVP-architected core (`orc/core`, `orc/presenters`, `orc/view-types`). 

- **Type:** C++ / Qt6 cross-platform application
- **Build system:** CMake 3.20+ with vcpkg dependencies
- **Reproducible builds:** Nix (recommended) with flake.nix
- **Target platforms:** Linux (Flatpak), macOS (DMG), Windows (MSI)

## Architecture & Constraints

The project enforces **MVP (Model-View-Presenter) pattern** to keep layers decoupled. Do not bypass this:

- **Model/Core:** `orc/core/` — business logic, isolated from UI
- **Presenters:** `orc/presenters/` — translates core output to view models
- **View Types:** `orc/view-types/` — shared DTO-like structures
- **View:** `orc/gui/` and `orc/cli/` — consume presenters, never touch core directly

Run `ctest -R MVPArchitectureCheck` to validate boundaries before submitting PRs.

## Licensing & Legal Requirements

**Decode-Orc is licensed under GPLv3.** All dependencies and contributions must be compatible with GPLv3:

- **Permitted licenses:** GPLv3, GPLv2, LGPL, BSD, MIT, Apache 2.0, ISC, and similar permissive licenses
- **Incompatible licenses:** AGPL (stronger copyleft), proprietary/closed-source, SSPL
- **Check before adding:** Always verify a new dependency's license in its repository or LICENSE file before proposing a PR

**For contributors:**
- When adding a new dependency (via vcpkg.json or flake.nix), document its license
- If you're unsure about license compatibility, ask in the issue or PR
- vcpkg.json and flake.nix changes will be reviewed for license compliance during CI/CD

**License file location:** See [LICENSE](../LICENSE) in the repository root (GPLv3).

## Source Code Structure

**Repository root directory layout:**

```
decode-orc/
├── orc/                           # Main project directory (CMake target root)
│   ├── common/                    # Shared utilities (logging, file I/O, exceptions)
│   ├── core/                      # Core business logic (MVP Model layer)
│   │   ├── stages/                # Processing pipeline stages
│   │   ├── metadata/              # Metadata handling
│   │   └── [business logic]
│   ├── view-types/                # Shared DTO structures (MVP shared layer)
│   │   └── [data transfer objects used by presenters & UI]
│   ├── presenters/                # Presenter layer (MVP Presenter)
│   │   └── [translates core output to view models]
│   ├── cli/                       # Command-line interface
│   │   └── main.cpp
│   └── gui/                       # Qt6 graphical interface (optional, BUILD_GUI=ON)
│       └── [Qt widgets & dialogs]
├── orc-tests/                     # Unified test tree (compiled when test flags are enabled)
│   ├── core/
│   │   └── unit/                  # Unit tests for core module
│   └── gui/
│       └── unit/                  # GUI unit tests
├── cmake/                         # CMake build utilities
│   ├── check_mvp_architecture.sh  # MVP boundary validation script
│   ├── MVPEnforcement.cmake       # MVP constraint macros
│   └── [other build helpers]
├── .github/                       # GitHub-specific configuration
│   ├── copilot-instructions.md    # AI agent guidance (this file)
│   ├── workflows/                 # CI/CD pipelines
│   └── ISSUE_TEMPLATE/
├── external/                      # Third-party dependencies
│   └── ld-decode-tools/           # Legacy ld-decode reference (checked in; available locally)
├── docs/                          # Documentation
├── assets/                        # Images, logos
├── CMakeLists.txt                 # Top-level CMake config
├── CMakePresets.json              # CMake build presets for all platforms
├── flake.nix                      # Nix reproducible build configuration
├── vcpkg.json                     # Dependency manifest (vcpkg)
├── BUILD.md                       # Build instructions (detailed)
├── TESTING.md                     # Testing strategy & patterns
├── CONTRIBUTING.md                # Contribution guidelines
└── README.md                      # Project overview
```

**Important dependency handling notes:**

**Nix-based workflow (recommended):**
- `flake.nix` defines all external dependencies as flake inputs
  - `qtnodes` (Qt node editor) is fetched from GitHub during `nix develop` / `nix build`
  - `ezpwd-reed-solomon` headers are fetched from GitHub during `nix develop` / `nix build`
- No git submodules required; Nix handles everything automatically
- Simply run `nix develop` and dependencies are available
- When already working inside `nix develop`, if a temporary extra tool is needed to complete the task, prefer `nix shell` to pull that tool into the active workflow instead of installing it through the host system or assuming it already exists

**Non-Nix (CMake/vcpkg) workflow:**
- Dependencies are managed via `vcpkg.json` (manifest mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simoninns/decode-orc](https://github.com/simoninns/decode-orc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
