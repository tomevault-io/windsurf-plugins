---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Nix flake that creates a secure bubblewrap sandbox environment specifically designed for running Claude Code. The project uses a profile-based architecture to provide language-specific development environments with network isolation. The flake is designed to be easily imported and extended with custom profiles in other projects.

## Architecture

The flake defines sandboxed environments using:
- **Bubblewrap**: Provides process isolation and filesystem sandboxing
- **Nix**: Manages dependencies and creates reproducible environments
- **Claude Code**: Pre-installed and aliased with `--dangerously-skip-permissions`
- **Profile System**: Structured configuration for language-specific toolchains

Key components:
- `lib/sandbox/default.nix`: Core sandbox script generation
- `lib/default.nix`: Extensible API functions (mkSandbox, mkDevShell, deriveProfile)
- `lib/sandbox/profiles.nix`: Language-specific profile definitions
- `lib/proxy/`: HTTP proxy with allowlist functionality
- `flake.nix`: Main flake configuration and package exports

## Profile-Based Architecture

Profiles are structured configurations containing:
- `name`: Profile identifier
- `packages`: List of Nix packages to include
- `env`: Environment variables
- `preStartHooks`: Array of shell commands to execute at sandbox startup
- `args`: Additional bubblewrap arguments (for cache binds, etc.)
- `allowList`: List of domains allowed through HTTP proxy
- `customPrompt`: Custom system prompt for Claude Code

Base profile (`base`) provides core utilities, and other profiles extend it using `deriveProfile`.

## Extensible API

The flake exports functions for creating and customizing sandboxes:

### Core Functions
- `mkSandbox profile`: Create sandbox from profile specification
- `mkDevShell { packages, shellHook }`: Create extensible development shell
- `deriveProfile baseProfile extensions`: Extend existing profile
- `profiles`: Access to predefined language-specific profiles

### Profile Structure
```nix
{
  name = "profile-name";
  packages = with pkgs; [ tool1 tool2 ];
  env = { VAR = "value"; };
  preStartHooks = [  # optional
    ''export SECRET="$(cat /path/to/secret)"''
    ''echo "Setup complete"''
  ];
  args = [ "--ro-bind-try /cache /cache" ];
  allowList = ["api.example.com" "cdn.example.com"];  # optional
  customPrompt = "You are an expert in...";  # optional
}
```

## Available Profiles

### Language Development Profiles
- **base**: Core development tools (git, vim, ripgrep, etc.)
- **bare**: Minimal environment (bash, coreutils only)
- **nix**: Nix development (nix, alejandra) with `/nix` bind
- **go**: Go development with module cache binding
- **python**: Python development with pip/poetry/uv cache binding
- **rust**: Rust development with cargo cache binding
- **cpp**: C++ development with ccache binding
- **js**: JavaScript/TypeScript with npm/yarn/pnpm/bun cache binding
- **devops**: DevOps tools (docker, kubectl, terraform) with config binding

### Cache Management
Each profile automatically binds appropriate cache directories:
- Go: `~/go/pkg/mod`
- Python: `~/.cache/pip`, `~/.cache/pypoetry`, `~/.cache/uv`
- Rust: `~/.cargo/registry`, `~/.cargo/git`
- JavaScript: `~/.npm`, `~/.yarn`, `~/.bun`, pnpm store
- DevOps: `~/.kube`, `~/.aws`, `~/.cache/helm`, `~/.terraform.d`

## Commands

### Development Environment
```bash
# Enter development shell
nix develop
# or with direnv
direnv allow

# Run base sandbox in current directory
nix run

# Run sandbox in specific directory
nix run .#claude-sandbox [directory]
```

### Language-Specific Profiles
```bash
# Available profiles
nix run .#claude-sandbox-bare      # Minimal environment
nix run .#claude-sandbox-nix       # Nix development
nix run .#claude-sandbox-go        # Go development
nix run .#claude-sandbox-python    # Python development
nix run .#claude-sandbox-rust      # Rust development
nix run .#claude-sandbox-cpp       # C++ development
nix run .#claude-sandbox-js        # JavaScript/TypeScript
nix run .#claude-sandbox-devops    # DevOps tooling
```

### Inside the Sandbox
```bash
# Claude Code is aliased and ready to use
claude

# Available in all profiles:
# - Version control: git, jujutsu
# - Text processing: ripgrep, fd, jq, yq, less
# - File operations: tree, rsync, zip/unzip, tar, gzip
# - System tools: bash, coreutils, procps, which, file
# - Editor: vim with man pages
```

## Importing and Extending

### Basic Import
```nix
{
  inputs.bubblewrap-claude.url = "github:matgawin/bubblewrap-claude";

  outputs = {nixpkgs, bubblewrap-claude, ...}: let
    system = "x86_64-linux";
    bwLib = bubblewrap-claude.lib.${system};
  in {
    packages.${system}.my-sandbox = bwLib.mkSandbox {
      name = "my-project";
      packages = with pkgs; [ docker kubectl terraform ];
      env = { PROJECT_ENV = "development"; };
    };
  };
}
```

### Extending Existing Profiles
```nix
let
  customGoProfile = bwLib.deriveProfile bwLib.profiles.go {
    name = "go-web";
    packages = with pkgs; [ air templ ];
    env = { GO_ENV = "development"; };
    preStartHooks = [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matgawin/bubblewrap-claude](https://github.com/matgawin/bubblewrap-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
