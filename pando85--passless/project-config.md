---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Passless is a software FIDO2 authenticator emulator written in Rust. It creates a virtual FIDO2 security token that operates through the UHID (Userspace HID) kernel interface, allowing systems to authenticate using FIDO2/WebAuthn without physical hardware tokens.

**Key characteristics:**
- Uses the `keylib` crate for FIDO2/CTAP protocol implementation
- Operates as a platform authenticator with resident key (passkey) support
- Supports multiple storage backends: local filesystem, password-store (pass), and TPM 2.0
- Implements security hardening (mlock, core dump prevention)
- Desktop notifications for user verification prompts

## Development Commands

### Building
```bash
# Standard build
cargo build

# Release build
cargo build --release

# Full release with vendoring (for distribution)
make release
```

**Shell Completions:**
Shell completions for bash, zsh, fish, and elvish are automatically generated during build time via `build.rs`. The completions are placed in `target/*/build/passless-rs-*/out/completions/` and are installed by the PKGBUILD to the appropriate system directories:
- Bash: `/usr/share/bash-completion/completions/passless`
- Zsh: `/usr/share/zsh/site-functions/_passless`
- Fish: `/usr/share/fish/vendor_completions.d/passless.fish`
- Elvish: `/usr/share/elvish/lib/passless.elv`

### Testing
```bash
# Run unit tests
cargo test

# Run unit tests with linting
make test

# Run E2E tests (requires authenticator to be running or will auto-start)
make test-e2e
```

The E2E tests use environment variables:
- `PASSLESS_E2E_AUTO_ACCEPT_UV=1` - Auto-accept user verification (debug builds only)
- `PASSLESS_LOCAL_PATH=/tmp/passless/fido2` - Use temporary storage

### Linting and Formatting
```bash
# Format code
cargo fmt
# or
make fmt

# Check formatting
make fmt-check

# Run clippy
cargo clippy --all-targets --all-features -- -D warnings
# or
make clippy

# Auto-fix clippy issues
make clippy-fix

# Run all lints
make lint

# Auto-fix all lints
make lint-fix
```

### Pre-commit Hooks
```bash
# Install hooks
make pre-commit-install

# Run hooks manually
make pre-commit
```

### Running the Authenticator

The authenticator requires UHID kernel module and proper permissions:
```bash
# As root, setup UHID access
modprobe uhid
groupadd fido 2>/dev/null || true
usermod -a -G fido $USER
echo 'KERNEL=="uhid", GROUP="fido", MODE="0660"' > /etc/udev/rules.d/90-uinput.rules
udevadm control --reload-rules && udevadm trigger

# Run authenticator
cargo run

# With verbose logging
cargo run -- --verbose

# With custom backend
cargo run -- --backend-type pass
cargo run -- --backend-type local --local-path /tmp/passless
cargo run -- --backend-type tpm --tpm-tcti "device:/dev/tpmrm0"

# With swtpm (software TPM for testing)
cargo run -- --backend-type tpm --tpm-tcti "swtpm:path=$HOME/.local/run/swtpm-sock"
```

**TPM Backend Setup:**
For detailed TPM and swtpm setup instructions, see:
- `docs/TPM_SETUP.md` - Comprehensive TPM setup guide
- `docs/SWTPM_QUICK_START.md` - Quick swtpm reference

## Architecture

### Core Components

**Main Loop** (`src/main.rs:22-86`):
- Runs in `run_with_service()` function
- Reads CTAPHID packets from UHID device in 64-byte chunks
- Processes CBOR commands through `AuthenticatorService`
- Sends response packets back through UHID
- Periodically cleans up expired credential cache (every 5 seconds)

**AuthenticatorService** (`src/authenticator.rs`):
- Orchestrates FIDO2 operations using dependency-injected `CredentialStorage`
- Builds callbacks for credential operations (read, write, delete, iterate)
- Handles user presence/verification through notifications
- Wraps `keylib::Authenticator` with storage backend integration

**Storage Abstraction** (`src/storage/mod.rs`):
- `CredentialStorage` trait defines interface for all backends
- Key methods: `read_first()`, `read_next()`, `read()`, `write()`, `delete()`, `select_users()`
- Supports iteration with filtering (by ID, RP, or hash)
- Cache cleanup method for security (removes sensitive data from memory)

**Storage Implementations**:
- `LocalStorageAdapter` (`src/storage/local.rs`): JSON files in local directory with in-memory caching
- `PassStorageAdapter` (`src/storage/pass.rs`): Encrypted storage using password-store/GPG
  - Automatic git sync: If the password-store has a git remote configured, changes are automatically synced
  - Git pull on initialization: Ensures latest credentials are loaded
  - Git commit + push on write/delete: Automatically commits with descriptive messages and pushes to remote
  - Non-blocking: Git failures are logged as warnings and don't prevent credential operations
- `TpmStorageAdapter` (`src/storage/tpm.rs`): TPM 2.0 hardware-backed storage
  - Credentials sealed by TPM, can only be unsealed on the same machine
  - Uses `tss-esapi` crate for TPM operations
  - Supports hardware TPM and software TPM (swtpm) via TCTI configuration
  - Stores sealed credentials as files with `.tpm` extension
  - Uses owner hierarchy for sealing/unsealing operations

**CTAP Command Handlers** (`src/commands/`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pando85/passless](https://github.com/pando85/passless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
