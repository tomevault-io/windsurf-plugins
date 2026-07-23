---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`rules_haskell` provides Bazel build rules for Haskell projects. It extends Bazel with capabilities to build, test, and manage Haskell code at scale, supporting multiple GHC versions and both binary distributions and Nixpkgs-based toolchains.

## Build System

### Core Commands

Build, test, and lint commands:

```bash
# Build all targets
bazel build //...

# Run all tests
bazel test //...

# Build and run tests (separate workspaces)
bazel test //... && cd rules_haskell_tests && bazel test //...

# Format check (buildifier)
bazel run //buildifier && cd rules_haskell_tests && bazel run //buildifier

# Fix formatting
bazel run //buildifier:buildifier-fix && cd rules_haskell_tests && bazel run //buildifier:buildifier-fix

# Build documentation
bazel build //docs:api_html
bazel build //docs:guide_html

# Run coverage
bazel coverage //...
```

### Configuration Management

The repository uses a layered `.bazelrc` configuration:

- `.bazelrc` - imports other configs
- `.bazelrc.common` - shared settings, platform configs
- `.bazelrc.bzlmod` - bzlmod-specific settings
- `.bazelrc.local` - user-specific settings (gitignored)
- `.bazelrc.auth` - authentication for remote cache (gitignored)

Platform-specific configurations:
- `--config=linux-nixpkgs` - Linux with Nixpkgs toolchain
- `--config=macos-nixpkgs` - macOS with Nixpkgs toolchain
- `--config=linux-bindist` - Linux with GHC binary distribution
- `--config=macos-bindist` - macOS with GHC binary distribution
- `--config=windows-bindist` - Windows with GHC binary distribution
- `--config=ci` - CI-specific settings with remote cache

Add your preferred config to `.bazelrc.local`:
```
build --config=linux-nixpkgs
```

### Nix Development Environment

For Nixpkgs-based development (recommended on NixOS, optional elsewhere):

```bash
# Enter nix shell (provides Bazel, GHC, and all dependencies)
nix-shell --pure shell.nix

# Specify GHC version (default: 9.4.8)
nix-shell --pure shell.nix --argstr ghcVersion 9.6.5

# Without doc tools (faster)
nix-shell --pure shell.nix --arg docTools false
```

The nix shell provides: Bazel 6, GHC (version-specific), Stack, Go, Python3, JDK11, git, and various utilities.

### Module Structure

The repository is organized into three separate Bazel workspaces:

1. **`rules_haskell/` (root)** - Core rule definitions and implementation
   - `haskell/*.bzl` - Public rule APIs
   - `haskell/private/*.bzl` - Internal implementation details
   - `haskell/asterius/` - Asterius (Haskell-to-WebAssembly) support
   - `docs/` - Documentation source

2. **`rules_haskell_nix/`** - Nixpkgs integration and examples
   - Tests for Nixpkgs-specific functionality

3. **`rules_haskell_tests/`** - Comprehensive test suite
   - `tests/` - Test cases covering various scenarios
   - Tests must be run separately: `cd rules_haskell_tests && bazel test //...`

### Testing

Tag-based test filtering is critical for platform compatibility:

- `-requires_nix` - Exclude Nixpkgs-only tests when using bindists
- `-dont_test_on_darwin` - Exclude macOS-incompatible tests
- `-dont_test_on_windows` - Exclude Windows-incompatible tests
- `-dont_test_with_bindist` - Exclude tests incompatible with GHC bindists
- `-requires_threaded_rts` - Exclude tests needing threaded runtime
- `coverage-compatible` - Only tests that work with coverage

Tests are automatically filtered based on the selected config (`linux-bindist`, `macos-nixpkgs`, etc.).

### GHC Version Management

GHC versions are managed through:
- `haskell/private/ghc_ci.bzl` - Defines default GHC version
- `haskell/ghc_bindist.bzl` - Binary distribution support
- `stackage_snapshot.yaml` and version-specific variants (`stackage_snapshot_9.4.8.yaml`, etc.)

Supported GHC versions: 9.4.8 (default), 9.6.5, 9.8.2, 9.10.3

Set GHC version via environment variable:
```bash
export GHC_VERSION=9.6.5
bazel build //...
```

## Architecture

### Rule Implementation Pattern

Haskell rules follow a three-layer architecture:

1. **Public API** (`haskell/defs.bzl`)
   - User-facing macros and rules
   - Attribute definitions in `_haskell_common_attrs`
   - Exports: `haskell_library`, `haskell_binary`, `haskell_test`, `haskell_doc`, `haskell_repl`

2. **Implementation** (`haskell/private/haskell_impl.bzl`)
   - Core logic for compilation, linking, and packaging
   - Functions: `haskell_library_impl`, `haskell_binary_impl`, `haskell_test_impl`

3. **Actions** (`haskell/private/actions/*.bzl`)
   - Low-level build actions
   - `compile.bzl` - GHC compilation
   - `link.bzl` - Linking executables
   - `package.bzl` - Creating package databases
   - `info.bzl` - Extracting package information

### Provider System

Providers pass information between rules:

- `HaskellLibraryInfo` - Library metadata, exposed modules, dependencies
- `HaskellBinaryInfo` - Binary outputs
- `HaskellModuleInfo` - Experimental module-level information
- `HaskellCoverageInfo` - Code coverage data

### Toolchain Architecture

Toolchains (`haskell/toolchain.bzl`) define platform-specific GHC configurations:
- GHC version and tools (ghc, ghc-pkg, haddock, runghc)
- Compiler flags

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tweag/rules_haskell](https://github.com/tweag/rules_haskell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
