---
trigger: always_on
description: This is a native Linux backend daemon for Claude Desktop's Cowork feature. It reverse-engineers the Windows `cowork-svc.exe` (Go/Hyper-V) and implements the same JSON-over-Unix-socket protocol, but executes directly on the host (no VM).
---

# CLAUDE.md - Project Guidelines

## Project Overview

This is a native Linux backend daemon for Claude Desktop's Cowork feature. It reverse-engineers the Windows `cowork-svc.exe` (Go/Hyper-V) and implements the same JSON-over-Unix-socket protocol, but executes directly on the host (no VM).

**Language:** Pure Go, zero external dependencies (stdlib only).

**Binary:** `cowork-svc-linux`

**Socket:** `$XDG_RUNTIME_DIR/cowork-vm-service.sock` (falls back to `/tmp/` if `$XDG_RUNTIME_DIR` is unset).

**Session dirs:** `~/.local/share/claude-cowork/sessions/<name>/`

**Protocol:** 22 RPC methods over length-prefixed JSON (4-byte big-endian header, max 10 MB per message).

**Key constraint:** The upstream binary (`cowork-svc.exe`) is managed remotely by Anthropic and changes without notice. Every RPC method, parameter name, and protocol behavior can change between releases. This makes the project inherently fragile --- protocol documentation and handler code must be re-validated on each upstream update.

## Build & Run

```bash
# Build
make build

# Build for ARM64
make build-arm64

# Install (binary + systemd service)
sudo make install

# Run manually in debug mode
cowork-svc-linux -debug

# Run via systemd
systemctl --user start claude-cowork

# Lint
make lint

# Test
make test
```

## Key Files & Purposes

| File / Directory | Purpose |
|---|---|
| `main.go` | Entry point, flag parsing, socket path resolution |
| `native/backend.go` | Core native backend: VM lifecycle simulation, spawn with path remapping, mount handling, `--disallowedTools` stripping, `--brief` injection, `present_files` interception |
| `native/process.go` | Process management: binary resolution (3-stage fallback), stdout/stderr streaming, path remapping, skill prefix stripping, MCP control logging |
| `pipe/server.go` | Unix socket server, connection handling |
| `pipe/handlers.go` | RPC method dispatch, parameter parsing, all struct definitions with JSON tags |
| `pipe/protocol.go` | Wire protocol: `Request`/`Response` types, length-prefixed read/write (4-byte big-endian, max 10 MB) |
| `process/events.go` | Event type definitions (stdout, stderr, exit, apiReachability, error, startupStep) |
| `process/spawn.go` | Legacy process tracker (used by VM mode) |
| `vm/` | Dormant QEMU/KVM backend (manager, qemu, vsock, bundle, network) |
| `scripts/extract-cowork-svc.sh` | Downloads latest Claude Desktop, extracts `bin/` contents |
| `scripts/extract-vm-bundle.sh` | Downloads latest, extracts VM bundle (rootfs, vmlinuz, initrd, config) |
| `bin/` | Extracted upstream binaries (`cowork-svc.exe`, `app.asar`, locale files, icons) with `.version` file |
| `vm-bundle/` | Extracted VM bundle (rootfs.vhdx.zst, vmlinuz.zst, initrd.zst, config) with `.version` file |
| `Makefile` | Build automation (build, build-arm64, install, clean, lint, test) |
| `PKGBUILD` | Arch Linux AUR package definition |
| `flake.nix` + `packaging/nix/` | NixOS flake, module, and evaluation tests |
| `packaging/debian/` | `.deb` build scripts |
| `packaging/rpm/` | `.rpm` build scripts + repo infrastructure |
| `packaging/apt/` | APT repository infrastructure |
| `.upstream-version` | Committed upstream Claude Desktop version (used by CI version-check workflow) |
| `dist/` | Compiled binary + systemd service file |

## Upstream Reference Materials

- `bin/` --- Extracted from Claude Desktop Windows installer (`cowork-svc.exe` lives alongside `app.asar`, locale JSONs, icons)
- `vm-bundle/` --- VM images + config downloaded from Anthropic CDN
- Both directories have `.version` files tracking the Claude Desktop version they were extracted from
- Currently at version **1.3561.0**

## Version-Sensitive Artifacts

These files embed assumptions about upstream internals and **must be re-validated on every upstream release**:

| File | What's fragile | How to verify |
|---|---|---|
| `COWORK_RPC_PROTOCOL.md` | RPC method names, parameters, response shapes | Diff against new `cowork-svc.exe` behavior |
| `COWORK_VM_BUNDLE.md` | VM files, checksums, config format | Compare against new bundle contents |
| `COWORK_SVC_BINARY.md` | Binary behavior, startup sequence, flag handling | Run new binary and observe |
| `native/backend.go` | Spawn parameters, mount handling, path remapping | Test all session types (chat, code, dispatch) |
| `native/process.go` | Event types, output formats, binary resolution | Check new CLI flags and output |
| `pipe/handlers.go` | RPC method set, parameter structs | Add handlers for any new methods |
| `CHANGELOG.md` | Documents all changes per release | Update the `Unreleased` section with upstream changes |

**Rule of thumb:** If a handler references a specific RPC method or parameter name, it may be wrong after the next upstream release. Always verify against the actual protocol traffic.

**CHANGELOG.md:** Every upstream update or code change MUST be documented in `CHANGELOG.md` under the `## Unreleased` section. Use subsections `### Added`, `### Changed`, `### Fixed`, `### Removed` as appropriate. This file is the user-facing record of what changed and why.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickjaja/claude-cowork-service](https://github.com/patrickjaja/claude-cowork-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
