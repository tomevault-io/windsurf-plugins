---
trigger: always_on
description: Handles identity creation, message composition, encryption, and inbox display.
---

# Freenet Email – Agent Guide

## Overview

Decentralized email application built on Freenet. Uses Dioxus for the web UI,
WASM contracts for inbox storage, and Anti-Flood Tokens (AFT) for rate limiting.

## One-time setup

```bash
git config core.hooksPath .githooks   # Enable repo-local git hooks
```

The pre-commit hook blocks stray `.wasm` commits outside
`published-contract/` and requires `contract-id.txt` to be staged
alongside any WASM change.

## Quick Reference

### Commands

```bash
cargo make build                 # Full release build (UI + contracts)
cargo make dev                   # Local Dioxus dev server
cargo make dev-example           # Offline dev server (mock data, no node)
cargo make test                  # Run all tests
cargo make test-inbox            # Run inbox contract integration tests
cargo make clippy                # Lint
cargo make run-node              # Start local Freenet node

# Publishing pipeline (see "Publishing" section below)
cargo make publish-email-test    # Sandbox publish with committed test key
cargo make publish-email         # Production publish with uncommitted key
cargo make update-published-contract       # Refresh test snapshot
cargo make update-published-contract-prod  # Refresh production snapshot
cargo make publish-all           # End-to-end test publish (build → sign → publish)
cargo make publish-production    # End-to-end production publish
cargo make publish               # Alias for publish-email-test

# Release automation (see RELEASING.md)
scripts/generate-production-key.sh       # One-time: generate prod ed25519 key
scripts/release.sh <version>             # Full release: build → sign → publish → tag → push
scripts/smoke-test-production.sh <url>   # Playwright suite against deployed webapp
```

### Repository Structure

```
freenet-email/
├── common/                  # Shared types (freenet-email-core)
├── contracts/
│   ├── inbox/               # Email inbox contract (WASM)
│   └── web-container/       # Web container contract (WASM)
├── ui/                      # Dioxus web UI
│   └── src/
│       ├── lib.rs           # Entry point, WEB_CONTAINER_CONTRACT_ID embed
│       ├── app.rs           # Main component, inbox UI
│       ├── app/login.rs     # Identity management UI
│       ├── api.rs           # WebSocket communication with Freenet node
│       ├── aft.rs           # Anti-Flood Token management
│       ├── inbox.rs         # Inbox state & message encryption
│       ├── log.rs           # Logging abstraction
│       └── test_util.rs     # Test helpers
├── modules/                 # Vendored dependencies
│   ├── antiflood-tokens/
│   │   └── interfaces/      # freenet-aft-interface
│   └── identity-management/ # Identity delegate
├── tools/
│   └── web-container-sign/  # ed25519 signer binary (our web-container-tool)
├── test-contract/           # Committed test keys (identity + web-container)
├── published-contract/      # Committed signed-webapp snapshot
├── Cargo.toml               # Workspace root
└── Makefile.toml            # cargo-make build system
```

### Key Dependencies

| Dependency | Purpose |
|-----------|---------|
| `freenet-stdlib` | Freenet contract/delegate SDK |
| `dioxus` | Web UI framework (WASM) |
| `ml-kem` | ML-KEM-768 (NIST FIPS 203) post-quantum key encapsulation for message encryption |
| `ml-dsa` | ML-DSA-65 (NIST FIPS 204) post-quantum signatures for inbox state deltas and AFT |
| `chacha20poly1305` | Symmetric encryption for message content (key from ML-KEM) |
| `freenet-aft-interface` | Anti-Flood Token protocol |
| `identity-management` | Identity delegate for alias management |

### Architecture

- **Inbox Contract**: Stores encrypted messages on Freenet. ML-DSA-65 signatures
  verify ownership of state deltas. Messages are gated by AFT tokens to prevent spam.
- **Web Container**: Minimal contract that hosts the compiled Dioxus UI as a
  Freenet webapp.
- **UI**: Dioxus WASM app communicating with a local Freenet node via WebSocket.
  Handles identity creation, message composition, encryption, and inbox display.
- **AFT Integration**: Each sent message requires a token from the Anti-Flood
  Token system, preventing spam while preserving sender privacy.

### Feature flag matrix

The UI crate at `ui/Cargo.toml` exposes four features that compose to
produce different builds. The cell shows whether that combination is a
supported build target and what it's used for.

| Flag          | Purpose                                                                                  |
|---------------|------------------------------------------------------------------------------------------|
| `use-node`    | Default. Enables the WebSocket bridge to a local Freenet node and all contract calls.    |
| `example-data`| Seeds the UI with two mock identities (`address1`, `address2`) and mock inboxes.          |
| `no-sync`     | Disables the WebSocket bridge entirely. Must be combined with `example-data` to be useful.|
| `contract`    | (inbox crate, not ui) Enables the inbox contract's `ContractInterface` impl.             |

**Supported combinations:**

| Build                                                        | What it's for                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freenet/mail](https://github.com/freenet/mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
