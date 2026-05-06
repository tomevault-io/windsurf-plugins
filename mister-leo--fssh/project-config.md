---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

fssh is a macOS SSH private key manager that provides Touch ID or OTP (One-Time Password) authentication for SSH connections. It acts as an SSH agent that securely stores and manages encrypted private keys, eliminating the need to enter passphrases repeatedly while maintaining security.

**Core Purpose**: Replace plaintext SSH keys with encrypted storage, unlock via biometric (Touch ID) or TOTP, and provide an interactive shell for managing SSH hosts.

## Build and Development Commands

```bash
# Build the binary
go build ./cmd/fssh

# Install to /usr/local/bin
go build -o /usr/local/bin/fssh ./cmd/fssh

# Run tests (if any)
go test ./...

# Initialize master key (Touch ID mode - default)
./fssh init

# Initialize with OTP mode (for devices without Touch ID)
./fssh init --mode otp --seed-unlock-ttl 3600

# Import SSH private key
./fssh import -alias mykey -file ~/.ssh/id_rsa --ask-passphrase

# List imported keys
./fssh list

# Start SSH agent (secure mode, Touch ID/OTP per signature)
./fssh agent --unlock-ttl-seconds 600

# Start SSH agent (convenience mode, decrypt all on startup)
./fssh agent --require-touch-id-per-sign=false

# Export private key
./fssh export -alias mykey -out /path/to/key.pem --ask-passphrase

# Interactive shell for SSH connections
./fssh shell
# or simply
./fssh

# Remove a key
./fssh remove -alias mykey

# Rotate master key and re-encrypt all keys
./fssh rekey

# Check system status
./fssh status

# Generate SSH config entries
./fssh config-gen

# Align sshd config for RSA-SHA2 support
./fssh sshd-align
```

## Architecture Overview

### Authentication Layers

The project supports two authentication modes via a unified `AuthProvider` interface:

1. **Touch ID Mode** (default for macOS with Touch ID):
   - Master key stored in macOS Keychain
   - Biometric unlock via `internal/macos/touchid_darwin.go`
   - Hardware-backed security

2. **OTP Mode** (for devices without Touch ID):
   - Master key derived from password-encrypted OTP seed
   - TOTP verification (RFC 6238) via authenticator apps
   - Seed cached with TTL, supports recovery codes
   - Configuration stored in `~/.fssh/otp/config.enc`

Mode selection is stored in `~/.fssh/auth_mode.json` and determined by `internal/auth/auth.go:GetAuthProvider()`.

### Encryption Architecture

**Master Key Flow**:
```
User Auth (Touch ID/OTP) → Master Key → HKDF per-file key → AES-256-GCM → Encrypted Private Key
```

- **Master Key**: 32-byte key from Keychain (Touch ID) or password-derived (OTP)
- **Per-file Keys**: Derived using HKDF with unique salt per private key
- **Encryption**: AES-256-GCM with unique nonce, fingerprint as AAD
- **Storage**: JSON files in `~/.fssh/keys/*.enc`

See `internal/crypt/crypt.go` for crypto primitives and `internal/store/store.go` for key storage.

### SSH Agent Implementation

**Two Operating Modes**:

1. **Secure Mode** (`require_touch_id_per_sign: true`):
   - Implemented in `internal/agent/secure_agent.go`
   - Decrypts keys on-demand per SSH signature
   - Optional TTL cache for master key unlock
   - Every signature triggers `AuthProvider.UnlockMasterKey()`

2. **Convenience Mode** (`require_touch_id_per_sign: false`):
   - Uses standard `golang.org/x/crypto/ssh/agent.Keyring`
   - Decrypts all keys once at agent startup
   - Keys kept in memory until agent stops

The agent implements the SSH agent protocol via `golang.org/x/crypto/ssh/agent.Agent` interface, supporting:
- RSA-SHA2-256/512 signatures (via `SignWithFlags`)
- Public key listing
- Agent extensions

### Interactive Shell

`cmd/fssh/shell.go` provides an interactive prompt powered by `github.com/peterh/liner`:
- Parses `~/.ssh/config` via `internal/sshconfig/sshconfig.go`
- Tab completion for host names, IPs, and numeric IDs
- Commands: `list`, `search <term>`, `connect <host>`, `help`, `exit`
- Non-command input defaults to SSH connection

### Configuration Management

**User Configuration** (`~/.fssh/config.json`):
```json
{
  "socket": "~/.fssh/agent.sock",
  "require_touch_id_per_sign": true,
  "unlock_ttl_seconds": 600,
  "log_level": "info",
  "log_format": "plain"
}
```

Loaded by `internal/config/config.go`, used as defaults for CLI flags.

**Auto-start**: `contrib/com.fssh.agent.plist` for macOS LaunchAgent.

## Key Code Locations

### Authentication
- `internal/auth/auth.go` - `AuthProvider` interface and mode selection
- `internal/auth/touchid.go` - Touch ID implementation
- `internal/auth/otp.go` - OTP implementation
- `internal/otp/totp.go` - TOTP generation/verification
- `internal/otp/config.go` - OTP seed encryption/storage
- `internal/otp/recovery.go` - Recovery code generation/validation

### Cryptography
- `internal/crypt/crypt.go` - HKDF, AES-GCM encryption/decryption
- `internal/store/store.go` - Private key record storage and retrieval
- `internal/keychain/keychain.go` - macOS Keychain integration

### SSH Agent
- `internal/agent/server.go` - Agent server startup and mode dispatch
- `internal/agent/secure_agent.go` - On-demand decryption agent

### CLI Commands
- `cmd/fssh/main.go` - Command dispatch and flag parsing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mister-leo/fssh](https://github.com/Mister-leo/fssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
