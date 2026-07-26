---
trigger: always_on
description: Handles nvm installation via curl/wget/git:
---

# nvm Coding Agent Instructions

This document provides guidance for AI coding agents when working with the Node Version Manager (nvm) codebase.

## Overview

nvm is a version manager for Node.js, implemented as a POSIX-compliant function that works across multiple shells (sh, dash, bash, ksh, zsh). The codebase is primarily written in shell script and emphasizes portability and compatibility.

### Core Architecture

- **Main script**: `nvm.sh` - Contains all core functionality and the main `nvm()` function
- **Installation script**: `install.sh` - Handles downloading and installing nvm itself
- **Execution wrapper**: `nvm-exec` - Allows running commands with specific Node.js versions
- **Bash completion**: `bash_completion` - Provides tab completion for bash users
- **Tests**: Comprehensive test suite in `test/` directory using the [urchin](https://www.npmjs.com/package/urchin) test framework

## Key Files and Their Purposes

### `nvm.sh`
The core functionality file containing:
- Main `nvm()` function (starts around line 3000)
- All internal helper functions (prefixed with `nvm_`)
- Command implementations for install, use, ls, etc.
- Shell compatibility logic
- POSIX compliance utilities

### `install.sh`
Handles nvm installation via curl/wget/git:
- Downloads nvm from GitHub
- Sets up directory structure
- Configures shell integration
- Supports both git clone and script download methods

### `nvm-exec`
Simple wrapper script that:
- Sources nvm.sh with `--no-use` flag
- Switches to specified Node version via `NODE_VERSION` env var or `.nvmrc`
- Executes the provided command with that Node version

## Top-Level nvm Commands and Internal Functions

### Core Commands

#### `nvm install [version]`
- **Internal functions**: `nvm_install_binary()`, `nvm_install_source()`, `nvm_download_artifact()`
- Downloads and installs specified Node.js version
- Automatically `nvm use`s that version after installation
- Supports LTS versions, version ranges, and built-in aliases (like `node`, `stable`) and user-defined aliases
- Can install from binary or compile from source
- When compiling from source, accepts additional arguments that are passed to the compilation task

#### `nvm use [version]`
- **Internal functions**: `nvm_resolve_alias()`, `nvm_version_path()`, `nvm_change_path()`
- Switches current shell to use specified Node.js version
- Updates PATH environment variable
- Supports `.nvmrc` file integration

#### `nvm ls [pattern]`
- **Internal functions**: `nvm_ls()`, `nvm_tree_contains_path()`
- Lists installed Node.js versions
- Supports pattern matching and filtering
- Shows current version and aliases

#### `nvm ls-remote [pattern]`
- **Internal functions**: `nvm_ls_remote()`, `nvm_download()`, `nvm_ls_remote_index_tab()`
- Lists available Node.js versions from nodejs.org and iojs.org, or the env-var-configured mirrors
- Supports LTS filtering and pattern matching
- Downloads version index on-demand

#### `nvm alias [name] [version]`
- **Internal functions**: `nvm_alias()`, `nvm_alias_path()`
- Creates text files containing the mapped version, named as the alias name
- Special aliases: `default`, `node`, `iojs`, `stable`, `unstable` (note: `stable` and `unstable` are deprecated, from node's pre-v1 release plan)
- Stored in `$NVM_DIR/alias/` directory

#### `nvm current`
- **Internal functions**: `nvm_ls_current()`
- Shows currently active Node.js version
- Returns "system" if using system Node.js

#### `nvm which [version]`
- **Internal functions**: `nvm_version_path()`, `nvm_resolve_alias()`
- Shows path to specified Node.js version
- Resolves aliases and version strings

### Utility Commands

#### `nvm cache clear|dir`
- Cache management for downloaded binaries and source code
- Clears or shows cache directory path

#### `nvm debug`
- Diagnostic information for troubleshooting
- Shows environment, tool versions, and paths

#### `nvm deactivate`
- Removes nvm modifications from current shell
- Restores original PATH

#### `nvm unload`
- Completely removes nvm from shell environment
- Unsets all nvm functions and variables

### Internal Function Categories

#### Version Resolution
- `nvm_resolve_alias()` - Resolves aliases to version numbers
- `nvm_version()` - Finds best matching local version
- `nvm_remote_version()` - Finds best matching remote version
- `nvm_normalize_version()` - Standardizes version strings
- `nvm_version_greater()` - Compares version numbers
- `nvm_version_greater_than_or_equal_to()` - Version comparison with equality
- `nvm_get_latest()` - Gets latest version from a list

#### Installation Helpers
- `nvm_install_binary()` - Downloads and installs precompiled binaries
- `nvm_install_source()` - Compiles Node.js from source
- `nvm_download_artifact()` - Downloads tarballs or binaries
- `nvm_compute_checksum()` - Verifies download integrity
- `nvm_checksum()` - Checksum verification wrapper
- `nvm_get_mirror()` - Gets appropriate download mirror
- `nvm_get_arch()` - Determines system architecture

#### Path Management
- `nvm_change_path()` - Updates PATH for version switching
- `nvm_strip_path()` - Removes nvm paths from PATH
- `nvm_version_path()` - Gets installation path for version
- `nvm_version_dir()` - Gets version directory name
- `nvm_prepend_path()` - Safely prepends to PATH


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nvm-sh/nvm](https://github.com/nvm-sh/nvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
