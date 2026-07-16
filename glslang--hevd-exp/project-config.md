---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Rust workspace of Windows kernel exploit proof-of-concepts targeting [HackSys Extreme Vulnerable Driver (HEVD)](https://github.com/hacksysteam/HackSysExtremeVulnerableDriver). All exploits target Windows 11 and depend on the shared [`win-kexp`](https://github.com/glslang/win-kexp) library for Windows API wrappers, shellcode, and ROP utilities.

**Build target: Windows x86-64 only.** These exploits cannot be built or run on Linux.

## Commands

```bash
# Check formatting (CI requirement)
cargo fmt --all --verbose -- --check

# Auto-fix formatting
cargo fmt --all

# Build all exploits
cargo build --verbose

# Build a specific exploit
cargo build -p stack_buffer_overflow_token_stealing

# Run tests
cargo test --verbose

# Update dependencies
cargo update
```

CI also requires MSBuild and MASM (`glslang/setup-masm`) because `win-kexp` contains inline assembly that must be assembled with MASM.

## CI / Automation

### GitHub Actions (`.github/workflows/ci.yml`)

Triggers on push/PR to `main` and manual dispatch. Runs on `windows-latest` with Rust stable. Steps in order:

1. Checkout (`actions/checkout@v6`)
2. Setup MSBuild (`microsoft/setup-msbuild@v3`)
3. Setup MASM (`glslang/setup-masm@v1`)
4. Install Rust stable (`rustup update stable && rustup default stable`)
5. Cache Cargo registry and build artifacts (`Swatinem/rust-cache@v2`)
6. `cargo fmt --all --verbose -- --check` — formatting gate; must pass
7. `cargo update` — refresh dependencies
8. `cargo build --verbose`
9. `cargo test --verbose`

### Dependabot (`.github/dependabot.yml`)

- **Cargo**: weekly on Monday 09:00 UTC, limit 10 PRs, label `dependencies`/`rust`, prefix `cargo`
- **GitHub Actions**: weekly on Monday 09:00 UTC, limit 5 PRs, label `dependencies`/`github-actions`, prefix `ci`
- Reviewer/assignee: `glslang`

## Architecture

### Workspace Layout

Each subdirectory is an independent exploit crate with a single `src/main.rs`. All crates share a single external dependency: `win-kexp` (fetched from GitHub, no version pin — always latest `main`).

| Crate | Vulnerability | Technique | Mitigations Bypassed |
|---|---|---|---|
| `stack_buffer_overflow_token_stealing` | Stack buffer overflow | Token stealing shellcode | None (SMEP/KPTI disabled) |
| `stack_buffer_overflow_acl_edit` | Stack buffer overflow | ACL edit + process injection into winlogon | None (SMEP/KPTI disabled) |
| `stack_buffer_overflow_token_stealing_smep_no_kvashadow` | Stack buffer overflow | Token stealing + ROP SMEP disable via CR4 | SMEP (no KVA Shadow) |
| `stack_buffer_overflow_token_stealing_smep_no_kvashadow_pte` | Stack buffer overflow | Token stealing + PTE-based SMEP disable via ROP | SMEP (no KVA Shadow) |
| `buffer_overflow_non_paged_pool_nx` | UAF / Non-Paged Pool NX | Pool normalization primitive | — |
| `memory_disclosure_non_paged_pool_nx_named_pipe` | Memory disclosure + UAF | NamedPipe pool spray, heap grooming, pointer leak | — |

### The `win-kexp` Library

All exploit logic delegated to utility code lives in `win-kexp`. Key modules used across exploits:

- **`win32k`** — Windows kernel interaction wrappers: `get_device_handle`, `io_device_control`, `allocate_shellcode`, `get_ntoskrnl_base_address`, `load_library_no_resolve`, `allocate_memory`, `lock_memory`, `close_handle`, `create_cmd_process`; constants `FILE_ANY_ACCESS`, `FILE_DEVICE_UNKNOWN`, `METHOD_NEITHER`, `MEM_COMMIT`, `MEM_RESERVE`, `PAGE_EXECUTE_READWRITE`, `HANDLE`
- **`shellcode`** — Pre-built kernel shellcode blobs: `token_stealing_shellcode`, `token_stealing_shellcode_smep_no_kvashadow`, `token_stealing_shellcode_smep_no_kvashadow_pte`, `acl_edit_shellcode`, `spawn_cmd_shellcode`
- **`rop`** — ROP gadget scanning: `get_executable_sections`, `find_gadget_offset` (scans ntoskrnl sections loaded in userspace via `load_library_no_resolve`)
- **`process`** — `inject_shellcode_to_target_process` (classic remote thread injection into a named process)
- **`pool`** — `AnonymousPipe` RAII wrapper for pool spray primitives
- **`util`** — `bytes_to_hex_string` (formats a raw pointer + length as a hex string for diagnostic output)
- **Macros** — `IOCTL!` (computes IOCTL code from function code), `CTL_CODE!`, `create_rop_chain!` (builds a byte `Vec` from addresses + `base_offset` padding), `concat_rop_chain_to_buffer!` (writes multiple chains into a mutable slice)

## IOCTL Codes

All exploits define IOCTL constants at the top of `main.rs` using the `IOCTL!` macro:

```rust
const HEVD_IOCTL_BUFFER_OVERFLOW_STACK: u32                    = IOCTL!(0x800);
const HEVD_IOCTL_MEMORY_DISCLOSURE_NON_PAGED_POOL_NX: u32      = IOCTL!(0x813);
const HEVD_IOCTL_ALLOCATE_UAF_OBJECT_NON_PAGED_POOL_NX: u32    = IOCTL!(0x814);
```

## Common Patterns

**Device path** — All exploits open the same device: `r"\\.\HackSysExtremeVulnerableDriver\0"`

**Stack overflow trigger** — The HEVD stack buffer overflow return address is always at offset `0x818`. The payload buffer is `0x818` bytes of `0x41` filler followed by the 8-byte little-endian address:

```rust
let user_buffer: Vec<u8> = iter::repeat_n(0x41u8, 0x818)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glslang/hevd-exp](https://github.com/glslang/hevd-exp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
