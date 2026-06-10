---
trigger: always_on
description: **usethis** is a CLI tool that automates Python project setup and development tasks. It declaratively adds, removes, and configures popular Python tools (uv, Ruff, pytest, pre-commit, MkDocs, etc.) in existing projects without breaking configuration. The tool provides detailed ✔/☐/ℹ messages about what it automated and what users need to do next. See the README.md and docs/ for more details.
---

# Copilot Instructions

## Project Overview

**usethis** is a CLI tool that automates Python project setup and development tasks. It declaratively adds, removes, and configures popular Python tools (uv, Ruff, pytest, pre-commit, MkDocs, etc.) in existing projects without breaking configuration. The tool provides detailed ✔/☐/ℹ messages about what it automated and what users need to do next. See the README.md and docs/ for more details.

## Module Structure

<!-- sync:docs/module-tree.txt -->

```text
usethis                           # usethis: Automatically manage Python tooling and configuration: linters, formatters, and more.
├── __main__                      # The CLI application for usethis.
├── _config                       # Global configuration state for usethis.
├── _config_file                  # Context managers for coordinated configuration file I/O.
├── _console                      # Console output helpers for styled and structured printing.
├── _deps                         # Dependency management operations for project dependency groups.
├── _fallback                     # Central module for hard-coded fallback version constants.
├── _init                         # Project initialization and build system setup.
├── _subprocess                   # Subprocess invocation utilities.
├── errors                        # Custom errors for the usethis package.
├── _backend                      # Backend dispatch and tool-specific backend implementations.
│   ├── dispatch                  # Backend selection and dispatch logic.
│   ├── poetry                    # Poetry backend implementation.
│   │   ├── available             # Check whether the Poetry CLI is available.
│   │   ├── call                  # Subprocess wrappers for invoking Poetry commands.
│   │   ├── deps                  # Dependency group operations via the Poetry backend.
│   │   ├── detect                # Detection of Poetry usage in a project.
│   │   ├── errors                # Error types for the Poetry backend.
│   │   └── init                  # Project initialization via Poetry.
│   └── uv                        # uv backend implementation.
│       ├── available             # Check whether the uv CLI is available.
│       ├── call                  # Subprocess wrappers for invoking uv commands.
│       ├── deps                  # Dependency group operations via the uv backend.
│       ├── detect                # Detection of uv usage in a project.
│       ├── errors                # Error types for the uv backend.
│       ├── init                  # Project initialization via uv.
│       ├── link_mode             # Symlink link-mode configuration for uv.
│       ├── lockfile              # Lock file creation and management for uv.
│       └── toml                  # Manager for the uv.toml configuration file.
├── _core                         # Core business logic for usethis commands.
│   ├── author                    # Author metadata management for pyproject.toml.
│   ├── badge                     # README badge generation and management.
│   ├── browse                    # Open project-related URLs in a browser.
│   ├── docstyle                  # Docstring style configuration.
│   ├── list                      # List tools and their usage status.
│   ├── readme                    # README file creation and management.
│   ├── rule                      # Linter rule selection and configuration.
│   ├── show                      # Display project information.
│   ├── status                    # Development status classifier management.
│   └── tool                      # Tool functions to add/remove tools to/from the project.
├── _detect                       # The detections module.
│   ├── pre_commit                # Detection of pre-commit usage in a project.
│   └── readme                    # Detection of README file presence.
├── _file                         # Configuration file reading, writing, and merging.
│   ├── dir                       # Project directory name utilities.
│   ├── manager                   # Base file manager classes for configuration file I/O.
│   ├── merge                     # Deep merge utilities for nested mappings.
│   ├── print_                    # Pretty-printing helpers for configuration file keys.
│   ├── types_                    # Shared type aliases for file operations.
│   ├── ini                       # INI file reading and writing.
│   │   ├── errors                # Error types for INI file operations.
│   │   └── io_                   # INI file I/O manager.
│   ├── pyproject_toml            # pyproject.toml file reading and writing.
│   │   ├── deps                  # Dependency extraction from pyproject.toml.
│   │   ├── errors                # Error types for pyproject.toml operations.
│   │   ├── io_                   # pyproject.toml file I/O manager.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usethis-python/usethis-python](https://github.com/usethis-python/usethis-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
