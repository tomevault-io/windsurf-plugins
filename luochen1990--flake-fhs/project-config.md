---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Flake FHS** (Flake Flake Hierarchy Standard) is a framework for Nix flakes that automatically generates flake outputs from a standardized directory structure, eliminating the need to write repetitive `flake.nix` boilerplate code.

## Core Architecture

### Directory Mapping System

The framework implements an automatic mapping from directory structure to flake outputs:

| Subdirectories (Aliases) | File Pattern | Special Files | Recursive | Generated Output | Nix Command |
|---|---|---|:---:|---|---|
| `packages` (`pkgs`) | `<name>.nix` or `<name>/package.nix` | `scope.nix` | ✅ | `packages.<system>.<name>` | `nix build .#<name>` |
| `nixosModules` (`modules`) | `<name>/...` | `options.nix`, `default.nix` | ✅ | `nixosModules.<name>` | - |
| `nixosConfigurations` (`hosts`) | `<name>/configuration.nix` | `default.nix` | ✅ | `nixosConfigurations.<name>` | `nixos-rebuild --flake .#<name>` |
| `apps` | `<name>.nix` or `<name>/package.nix` | `scope.nix` | ✅ | `apps.<system>.<name>` | `nix run .#<name>` |
| `devShells` (`shells`) | `<name>.nix` | `default.nix` | ✅ | `devShells.<system>.<name>` | `nix develop .#<name>` |
| `templates` | `<name>/` | `flake.nix` | ❌ | `templates.<name>` | `nix flake init --template <url>#<name>` |
| `lib` | `<name>.nix` | - | ✅ | `lib.<name>` | `nix eval .#lib.<name>` |
| `checks` | `<name>.nix` or `<name>/package.nix` | `scope.nix` | ✅ | `checks.<system>.<name>` | `nix flake check .#<name>` |

### Host Metadata (nixosConfigurations)

Host directories can contain a `default.nix` file that exports host-specific metadata. This is evaluated before NixOS modules to configure the globally shared `pkgs` instance accurately for that host.

```nix
# hosts/my-host/default.nix
{
  system = "x86_64-linux";
  nixpkgs = {
    config = { allowUnfree = true; cudaSupport = true; };
    overlays = [ /* overlays */ ];
  };
}
```

### Unified Package Model

The framework unifies the handling of `packages`, `apps`, and `checks` under a single **"Scoped Package Tree"** model.

- **Unified Entry**: Supports both single-file (`<name>.nix`) and directory-based (`<name>/package.nix`) definitions.
- **Encapsulation**: If a directory contains `package.nix`, it is treated exclusively as a package definition. Other `.nix` files in that directory are ignored by the automatic scanner (treated as internal helper files).
- **Unified Build**: All components are built using `callPackage`, enjoying automatic dependency injection from `pkgs`.
- **Unified Scoping**: `scope.nix` is supported in all hierarchies (`pkgs`, `apps`, `checks`) to customize dependencies or inject parameters.
- **Explicit Context**: The `scope.nix` function receives the full system context (`pkgs`, `self`, `inputs`, `system`, `lib`) as arguments, allowing users to explicitly inject them into the package scope if desired. Auto-injection is avoided to keep the default scope clean.

### Specific Behaviors

- **Apps**: Automatically converts the built package into an App structure (`{ type="app"; program="..."; }`) by inferring the main program (via `meta.mainProgram` or package name).
- **Checks**: Treated as packages that run tests during build. Access to `self` or `inputs` is available via function arguments.

### Package Scope System (callPackage)

The framework uses `callPackage` to build packages. You can customize the `callPackage` context (scope) via `scope.nix`.

- **File**: `<dir>/scope.nix` (Applies to current directory and subdirectories)
- **Mechanism**:
  - `package.nix` is built using `currentScope.callPackage`.
  - `scope.nix` modifies `currentScope` for its directory (and children).
- **Signature**: `{ pkgs, inputs, ... }: { scope = ...; args = ...; }`
  - **scope** (Optional): The base package set (e.g., `pkgs.pythonPackages`) to use for `callPackage`.
    - If provided: **Replaces** the parent scope.
    - If omitted: **Inherits** the parent scope.
  - **args** (Optional): Attributes to pass as the **second argument** to `callPackage`.
    - These are merged with inherited args from parent directories.
    - Useful for injecting dependencies or configuration into `package.nix`.
- **Granularity**: Works at both directory level (for groups of packages) and package level (sibling of `package.nix`).
- **Usage**: Essential for Python, Perl, and other language-specific package sets, or for injecting parameters into packages.


### Key Components

- **lib/**: Core utility library with Haskell-inspired functional programming patterns
  - `lib/flake-fhs.nix`: Entry point wrapper for `mkFlake`
  - `lib/fhs-core.nix`: Core implementation (`mkFlakeCore`)
  - `lib/fhs-modules.nix`: Module system logic and output generation
  - `lib/fhs-pkgs.nix`: Package loading logic
  - `lib/fhs-lib.nix`: Library preparation and recursive loader
  - `lib/fhs-config.nix`: Configuration options
  - `lib/pkg-tools.nix`: Package helper utilities
  - `lib/dict.nix`, `lib/list.nix`, `lib/file.nix`: Fundamental utilities

 - **templates/**: Project templates for different use cases
   - `default` *(recommended)*: Minimal template — only flake.nix, no pre-created directories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luochen1990/flake-fhs](https://github.com/luochen1990/flake-fhs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
