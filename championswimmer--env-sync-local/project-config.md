---
trigger: always_on
description: **env-sync** is a distributed secrets synchronization tool for local networks. It allows multiple machines to sync `.env` style secrets without a central server, using peer-to-peer architecture with mDNS discovery.
---

# AGENTS.md - Project Guide for LLM Coding Agents

## Project Overview

**env-sync** is a distributed secrets synchronization tool for local networks. It allows multiple machines to sync `.env` style secrets without a central server, using peer-to-peer architecture with mDNS discovery.

**Current Version**: v3.0 - Security Mode Overhaul with Three Operation Modes

## Architecture

### Core Philosophy
- **Distributed**: No master server, all machines are equal
- **Zero Configuration**: New machines auto-discover without touching existing ones (in trusted-owner mode)
- **Local Network Only**: Uses mDNS/Bonjour for discovery
- **Eventually Consistent**: Syncs on shell startup, cron (30min), or manual trigger
- **Explicit Security Models**: Three distinct modes for different trust scenarios

### Three Security Modes (v3.0+)

#### Mode A: `dev-plaintext-http` (Debug Only)
- **Storage**: Plaintext
- **Transport**: Plaintext HTTP
- **Use Case**: Local debugging only
- **Security**: None - displays prominent warnings

#### Mode B: `trusted-owner-ssh` (Same Owner - Default)
- **Storage**: Plaintext (optional AGE encryption)
- **Transport**: SCP/SSH
- **Use Case**: All devices belong to one user, mutually trusted
- **Onboarding**: Zero-touch - new machines need only SSH access to one peer
- **Security**: SSH provides encrypted transport and authentication

#### Mode C: `secure-peer` (Cross-Owner Collaboration)
- **Storage**: AGE encrypted (mandatory)
- **Transport**: HTTPS with mTLS (mutual TLS)
- **Use Case**: Different owners sharing secrets without shell access
- **Onboarding**: Invitation-based with explicit approval
- **Security**: mTLS for authentication, AGE for at-rest encryption, explicit authorization

### Mode-Specific Sync Strategy

**Trusted-Owner Mode:**
1. Discovery: Find peers via mDNS, filter by SSH reachability
2. Fetch: Download secrets via SCP/SSH
3. Compare: Check versions and timestamps
4. Merge: Combine changes (if needed)
5. Save: Store file (plaintext or encrypted based on config)
6. Backup: Always backup before overwriting (keep last 5)

**Secure-Peer Mode:**
1. Discovery: Find peers via mDNS
2. Authentication: Establish mTLS connection
3. Authorization: Check peer approval status
4. Fetch: Download encrypted secrets via HTTPS
5. Decrypt: Decrypt using local AGE private key
6. Compare: Check versions and timestamps
7. Merge: Combine changes (per-key timestamps)
8. Re-encrypt: Encrypt to all known recipient keys
9. Backup: Create backup before modification
10. Update: Replace local file and update metadata

## File Structure

```
env.sync.local/
├── CHANGELOG.md               # Version history
├── README.md                  # User documentation
├── AGENTS.md                  # This file - internal dev documentation
├── docs/                      # Detailed documentation
│   ├── USAGE.md               # Complete usage guide
│   ├── SECURITY-MODES.md      # Security mode details
│   └── INSTALLATION.md        # Installation instructions
├── install.sh                 # Installation script
├── Makefile                   # Build automation
├── src/                       # Go source code (v3.0)
│   ├── cmd/env-sync/          # Main entry point
│   │   └── main.go
│   ├── internal/              # Internal packages
│   │   ├── cli/               # CLI interface and command routing
│   │   ├── mode/              # Mode management (NEW in v3.0)
│   │   ├── sync/              # Sync logic (mode-aware)
│   │   ├── discovery/         # mDNS peer discovery
│   │   ├── crypto/age/        # AGE encryption/decryption
│   │   ├── transport/         # Transport abstraction
│   │   │   ├── ssh/           # SCP transport (trusted-owner mode)
│   │   │   └── https/         # HTTPS+mTLS transport (secure-peer mode)
│   │   ├── server/            # HTTP/HTTPS server
│   │   │   └── secure/        # Secure peer server (mTLS)
│   │   ├── metadata/          # File metadata handling
│   │   ├── secrets/           # Secrets file management
│   │   ├── backup/            # Backup management
│   │   ├── keys/              # Key management (AGE + transport)
│   │   ├── peers/             # Peer registry (NEW in v3.0)
│   │   │   ├── registry.go    # Peer authorization management
│   │   │   └── trust/         # Certificate pinning
│   │   ├── membership/        # Membership events (NEW in v3.0)
│   │   ├── config/            # Configuration
│   │   ├── logging/           # Logging utilities
│   │   └── cron/              # Cron job management
│   ├── go.mod                 # Go module definition
│   └── go.sum                 # Go module checksums
├── target/                    # Build output
│   └── env-sync               # Compiled Go binary
└── tests/                     # Integration tests (BATS)
    ├── bats/                  # BATS test files
    ├── docker/                # Docker test environment
    └── utils/                 # Test utilities
```

## Go Implementation (v3.0)

### Main Components

#### cmd/env-sync/main.go
**Purpose**: Entry point for the Go binary
**Key Features**:
- Parses command-line arguments
- Routes to appropriate CLI handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [championswimmer/env.sync.local](https://github.com/championswimmer/env.sync.local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
