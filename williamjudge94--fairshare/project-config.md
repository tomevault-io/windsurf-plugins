---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**fairshare** is a Rust-based systemd resource manager for multi-user Linux systems. It provides fair CPU and memory allocation management using systemd user slices, allowing users to request resources dynamically while preventing over-allocation. The system includes configurable CPU and memory reserves to ensure the operating system and background processes always have resources available.

## Installation

### Quick Install (Recommended)

```bash
# Download and run the installer (detects local build or downloads latest release)
sudo bash install.sh
```

The install script will:
- Detect if PolicyKit is installed (automatically installs it if missing on apt/dnf/pacman systems)
- Download the latest release binary or use local build if available
- Install the binary and wrapper script
- Set up PolicyKit policies
- Configure default resource limits

### Building and Installing Manually

```bash
# Build release binary
cargo build --release

# Install wrapper and binary (requires sudo)
sudo make release

# Ensure PolicyKit is installed (required)
# Debian/Ubuntu: sudo apt install policykit-1
# Fedora/RHEL: sudo dnf install polkit
# Arch Linux: sudo pacman -S polkit

# Setup admin defaults and PolicyKit policies (REQUIRED for user commands)
sudo fairshare admin setup --cpu 1 --mem 2 --cpu-reserve 2 --mem-reserve 4
```

### Installation Paths

- **Wrapper script**: `/usr/local/bin/fairshare` (what users run)
- **Real binary**: `/usr/local/libexec/fairshare-bin` (called by wrapper)
- **PolicyKit policies**: Installed via `admin setup` command
- **Systemd configuration**: `/etc/systemd/system/user-.slice.d/00-defaults.conf`

The wrapper script auto-detects the binary location, supporting both:
- Package installation: `/usr/libexec/fairshare-bin`
- Local installation: `/usr/local/libexec/fairshare-bin`

## Development Commands

### Building
- **Debug build**: `cargo build`
- **Release build**: `cargo build --release`
- **Install wrapper and binary**: `sudo make release` (installs to `/usr/local/bin/fairshare` and `/usr/local/libexec/fairshare-bin`)
- **Build release binaries for distribution**:
  - One-time setup: `sudo make setup-cross`
  - Build x86_64 only: `make compile-x86_64`
  - Build both architectures: `make compile-releases` (creates `releases/fairshare-x86_64` and `releases/fairshare-aarch64`)

### Testing
- **Run all tests**: `cargo test`
- **Run tests with output**: `cargo test -- --nocapture`
- **Run a specific test**: `cargo test test_cli_help`
- **Run integration tests only**: `cargo test --test integration_tests`
- **Run CLI tests only**: `cargo test --test cli_tests`

### Code Quality
- **Check code**: `cargo check`
- **Format code**: `cargo fmt`
- **Lint code**: `cargo clippy`

### Running
- **Show help**: `cargo run -- --help`
- **Show status**: `fairshare status` (wrapper handles pkexec automatically)
- **Request resources**: `fairshare request --cpu 4 --mem 8`
- **Show user info**: `fairshare info`
- **Release resources**: `fairshare release`
- **Admin setup** (requires root): `sudo fairshare admin setup --cpu 1 --mem 2 --cpu-reserve 2 --mem-reserve 4`
- **Admin set user resources** (requires root): `sudo fairshare admin set-user --user username --cpu 4 --mem 8` (set resources for a specific user, even if signed out)

**Note**: Regular user commands automatically use pkexec via the wrapper script at `/usr/local/bin/fairshare`. The real binary is at `/usr/local/libexec/fairshare-bin`. Admin commands require `sudo`.

## Architecture Overview

### Wrapper Architecture

The user-facing `fairshare` command is a shell script wrapper that:
1. Auto-detects binary location (supports package and local installation)
2. Detects admin commands (first arg is "admin") → executes directly (requires sudo)
3. Regular user commands → transparently calls `pkexec /usr/local/libexec/fairshare-bin`
4. Provides a simple UX without requiring users to type `pkexec`

This pattern is used by many system tools for privilege escalation (e.g., `systemctl`, `nmcli`).

### Module Structure

The codebase is organized into four main modules:

1. **`src/main.rs`** - Entry point that routes commands to appropriate handlers
2. **`src/cli.rs`** - Command-line interface definitions using `clap` with validation constraints:
   - CPU range: 1-1000 cores
   - Memory range: 1-10000 GB
3. **`src/system.rs`** - System information gathering and resource availability checking
4. **`src/systemd.rs`** - Systemd interaction for applying/reverting resource limits

### Core Data Flow

1. **Wrapper Detection** (shell script): Detects command type and invokes pkexec for user commands
2. **Command Parsing** (`cli.rs`): Clap validates input bounds before execution
3. **Resource Validation** (`system.rs`): Checks if requested resources are available
4. **Systemd Configuration** (`systemd.rs`): Applies limits via `systemctl set-property` or reverts via `systemctl revert`

### Key Functions

#### System Information (`system.rs`)
- `get_system_totals()` - Returns total system CPU and memory (uses `sysinfo` crate)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WilliamJudge94/fairshare](https://github.com/WilliamJudge94/fairshare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
