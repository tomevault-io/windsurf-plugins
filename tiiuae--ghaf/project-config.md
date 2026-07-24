---
trigger: always_on
description: **Ghaf Framework** is a Nix-based open-source security framework for enhancing security through compartmentalization on edge devices. It creates secure images for various hardware platforms (x86, ARM, RISC-V) using NixOS.
---

# Ghaf Framework Development Instructions

**Ghaf Framework** is a Nix-based open-source security framework for enhancing security through compartmentalization on edge devices. It creates secure images for various hardware platforms (x86, ARM, RISC-V) using NixOS.

**CRITICAL: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Tool Initialization

When starting a new session, **ALWAYS initialize these tools first** before proceeding with any tasks:

### Serena (Code Intelligence)

**REQUIRED**: Initialize Serena MCP for semantic code navigation and intelligent editing.

Start by activating the project with this command:

```
#serena activate project
```

This invokes the Serena MCP server for code intelligence. The `#serena` prefix is required to access the MCP tools.

**Key Serena capabilities**:
- Semantic code search and navigation
- Symbol-level understanding (find_symbol, get_symbols_overview)
- Intelligent code editing (replace_symbol_body, insert_after_symbol)
- Cross-reference analysis (find_referencing_symbols)
- Memory system for project context

**When to use `#serena` commands**:
- Understanding code structure and relationships
- Finding specific functions, classes, or modules
- Making precise code modifications
- Analyzing dependencies between components
- Any task requiring code comprehension

All Serena commands must be prefixed with `#serena` to invoke the MCP server.

### Context7 (Documentation Intelligence)

**REQUIRED**: Use Context7 for up-to-date library documentation:

**Available Context7 libraries**:

- `/NixOS/nixos` - NixOS system configuration options
- `/NixOS/nixpkgs` - Nixpkgs package set and functions
- `/NixOS/nix` - Nix language and package manager
- `/nix-community/home-manager` - Home Manager user environment management
- `/Mic92/sops-nix` - SOPS secrets management for NixOS
- `/numtide/flake-utils` - Flake utility functions

**Note**: Always resolve library IDs first unless you know the exact Context7-compatible ID format.

## Project Overview

Ghaf Framework is a security-focused operating system framework that:
- **Compartmentalizes** applications and services into isolated VMs
- **Supports multiple architectures**: x86_64, aarch64 (ARM), riscv64
- **Targets diverse hardware**: From laptops to embedded devices (Jetson, i.MX)
- **Uses NixOS**: Declarative, reproducible, and immutable system configuration
- **Implements security by design**: Inter-VM communication via GIVC, minimal attack surface

### Architecture

The configuration uses a modular architecture:
- **`modules/`** - NixOS system modules
  - **`common/`** - Base configurations (networking, security, services, users, logging, etc.)
  - **`hardware/`** - Hardware-specific configurations (x86_64-generic, aarch64, passthrough)
  - **`desktop/`** - Desktop environment configurations
  - **`microvm/`** - MicroVM configurations for compartmentalization
  - **`givc/`** - Ghaf Inter-VM Communication library
  - **`development/`** - Debug tools, SSH, testing utilities
  - **`reference/`**, **`profiles/`**, **`partitioning/`** - Additional configs
- **`targets/`** - Hardware target configurations (vm, generic-x86_64, laptop, nvidia-jetson-orin, imx8mp-evk, etc.)
- **`packages/`** - Custom packages and overlays (pkgs-by-name structure)
- **`lib/`** - Library functions and build helpers
- **`docs/`** - Documentation (Astro Starlight framework)
- **`overlays/`** - Nix overlays
- **`nix/`** - Flake infrastructure
- **`tests/`** - Test configurations

## Prerequisites and Setup

### Initial Setup
- Install Nix package manager: `curl -L https://nixos.org/nix/install | sh`
- Enable flakes: `echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf`
- **CRITICAL**: For cross-compilation, set up an AArch64 remote builder: https://nixos.org/manual/nix/stable/advanced-topics/distributed-builds.html

### Development Environment
Enter the development shell to access all tools:
```bash
nix develop
```

This provides: treefmt, reuse, and all formatting tools configured for the project.

## Code Quality Standards

### **ALWAYS Strip Trailing Whitespace**
- **Automatically remove trailing whitespace** from any files you create or modify
- **Use sed command**: `sed -i 's/[[:space:]]*$//' filename` to clean files
- **Verify cleanup**: Ensure no trailing whitespace remains before staging changes
- **Project-wide consistency**: Maintain clean, professional code formatting standards

### **File Formatting Requirements**
- **All commits must be properly formatted** using treefmt before making a PR
- **Run formatting**: `nix fmt` or `nix fmt -- --fail-on-change`
- **License headers**: Always add proper SPDX license headers to new files
- **No trailing whitespace**: Clean, professional code standards

## Working Effectively

### Essential Build Commands
- View all available targets: `nix flake show` -- takes 30-60 seconds
- Build documentation: `nix build .#doc` -- takes 5-10 minutes. NEVER CANCEL. Set timeout to 15+ minutes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tiiuae/ghaf](https://github.com/tiiuae/ghaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
