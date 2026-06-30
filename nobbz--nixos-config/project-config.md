---
trigger: always_on
description: This is a **personal NixOS configuration repository** managing system and home-manager configurations for multiple hosts using Nix Flakes. The repository is ~12MB with 72 Nix files and uses a modular flake-parts architecture.
---

# Copilot Instructions for nixos-config

## Repository Overview

This is a **personal NixOS configuration repository** managing system and home-manager configurations for multiple hosts using Nix Flakes. The repository is ~12MB with 72 Nix files and uses a modular flake-parts architecture.

**⚠️ CRITICAL**: This repository contains hardware-specific configurations. Do NOT suggest building or deploying these configurations to other systems as they can render machines unbootable.

**Languages/Frameworks**: Nix (primary), CUE (workflow generation), Lua (AwesomeWM config), Shell scripts
**Target Systems**: x86_64-linux, aarch64-linux, aarch64-darwin
**Nix Version**: 2.32.1 (as specified in CI workflows)

## Build and Validation Commands

### Prerequisites
- Nix with flakes enabled must be installed
- CUE v0.14.2 required for workflow generation
- Tools available in dev shell: `nix develop` provides npins, sops, age, ssh-to-age, nil, alejandra, lua-language-server, cue

### Core Commands (Tested & Working)

**Formatting (ALWAYS run before committing)**:
```bash
nix fmt                    # Format all Nix files using alejandra
make fmt                   # Format both CUE and Nix files
```

**Linting**:
```bash
nix run --inputs-from . nixpkgs#statix -- check    # Lint Nix files
statix check -i packages/nodePackages/node-env.nix  # Alternative, ignores generated files
```
Note: `statix.toml` configures ignored paths and disabled checks. The `repeated_keys` check is disabled.

**Building Flake Outputs**:
```bash
nix flake check --keep-going    # Check all flake outputs (may take 10+ minutes)
nix flake show                  # Display all flake outputs
nix flake show --json           # JSON output for parsing
```

**Building Specific Outputs**:
```bash
nix build .#checks.x86_64-linux.<check-name> --no-link
nix build .#packages.x86_64-linux.<package-name>
nix build .#nixosConfigurations.mimas.config.system.build.toplevel
```

**Workflow Generation** (when modifying `cicd/*.cue`):
```bash
make workflows              # Generate all workflows from CUE definitions
make check                  # Validate CUE matches generated YAML files
```
**IMPORTANT**: If you modify `cicd/*.cue` files, ALWAYS run `make workflows` to regenerate the YAML files before committing.

### Git Pre-commit Hooks
The repository uses `lefthook` for pre-commit checks:
- **Formatting check**: Runs `nix fmt -- --check` on staged `.nix` files
- **Linting**: Runs `statix check` on `.nix` files

To manually trigger these: Run the commands above before committing.

## CI/CD Pipeline

### Pull Request Checks (`.github/workflows/pull-check.yml`)
1. **generate_matrix**: Discovers all packages and checks from flake
2. **build_flake**: Builds all packages (except installer-iso) in parallel (max 5 concurrent)
3. **build_checks**: Builds all checks in parallel
4. **check_flake**: Runs `nix flake check --keep-going` (continue-on-error)

**Build Time**: Full CI can take 30-60+ minutes depending on cache hits. Uses cachix (nobbz cache) and nix-community cache.

### Generated Files Check (`.github/workflows/check-generated.yml`)
- Runs `make check` to verify CUE definitions match generated YAML files
- **CRITICAL**: This will FAIL if you modify `cicd/*.cue` without running `make workflows`

### Common CI Failure Patterns
1. **Formatting failures**: Run `nix fmt` before committing
2. **Statix failures**: Fix linting issues or add ignore patterns in `statix.toml`
3. **Generated files mismatch**: Run `make workflows` after modifying CUE files
4. **Build failures**: Test with `nix flake check` locally first
5. **Disk space issues**: Large builds use `wimpysworld/nothing-but-nix@main` action to free space

## Project Structure

### Root Directory Files
```
flake.nix           # Main flake entry point, imports all parts
flake.lock          # Locked dependencies (updated by bot)
Makefile            # CUE workflow generation and validation
lefthook.yml        # Pre-commit hook configuration
statix.toml         # Nix linter configuration
.envrc              # direnv configuration for dev shell
.sops.yaml          # Secret management with age keys
```

### Directory Layout
```
parts/              # Flake-parts modular configuration
├── auxiliary.nix           # Formatter, apps, devShell
├── system_configs.nix      # NixOS configuration generator (119 lines)
├── home_configs.nix        # Home-manager configuration generator (128 lines)
├── nixos_modules.nix       # NixOS module loader
├── home_modules.nix        # Home-manager module loader
└── module_helpers.nix      # Shared helpers for module loading

nixos/
├── configurations/         # Host-specific configurations
│   ├── default.nix        # Defines available hosts
│   ├── mimas.nix          # Primary host configuration
│   ├── hardware/*.nix     # Hardware-specific configs per host
│   └── bootloader/*.nix   # Bootloader configs per host
└── modules/               # Reusable NixOS modules
    ├── default.nix        # Module index
    ├── nix.nix           # Nix daemon configuration
    ├── switcher.nix      # NixOS rebuild wrapper

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NobbZ/nixos-config](https://github.com/NobbZ/nixos-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
