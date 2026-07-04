---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project Purpose

`windbg-tool` is a Windows-first Rust workspace for WinDbg automation. The `windbg-tool.exe` executable can run the `windbg-ttd` MCP server, keep long-lived TTD replay sessions in a local daemon, act as an agent-friendly CLI client, start DbgEng process servers, and install/update/launch WinDbg.

This project must use the Time Travel Debugging replay APIs, not the regular live-debugging `cdb` or DbgEng attach flow. DbgEng-related packages can be useful for symbols and debugger-platform support, but core replay should go through the TTD Replay API.

## Current Architecture

- Rust workspace root: `Cargo.toml`.
- TTD crate: `crates/windbg-ttd` (package name `windbg-ttd`).
- CLI application crate: `crates/windbg-tool` (package name `windbg-tool`, binary `windbg-tool.exe`).
- DbgEng helper crate: `crates/windbg-dbgeng`.
- WinDbg installer crate: `crates/windbg-install`.
- Developer workflow crate: `xtask`.
- Native C++ bridge scaffold: `native/ttd-replay-bridge`.
- Runtime/dependency helper script: `scripts/Get-TtdReplayRuntime.ps1`.
- Architecture notes: `docs/architecture.md`.

The intended layering is:

1. Rust MCP server over stdio using the official `rmcp` Rust MCP SDK, plus tool schemas, validation, session ids, and serialization.
2. Safe Rust replay facade for traces, cursors, positions, modules, threads, exceptions, registers, memory reads, and watchpoints.
3. Narrow C ABI C++ bridge over Microsoft's C++ TTD Replay API.

Do not bind Rust directly to TTD C++ vtables, STL helper types, or C++ ownership rules. Keep the native bridge as a small C ABI with opaque handles and plain data structs.

## Current Implementation State

The Rust MCP server uses `rmcp` for stdio MCP protocol handling, advertises tools, and can use the native bridge for trace pack/list enumeration, trace loading with trace-index selection, trace index status/stats/build operations, trace metadata, thread/module/exception/keyframe enumeration, cursor-local module snapshots, module and thread lifecycle event timelines, cursor creation, position get/set including TTD thread-scoped seeking, active-thread snapshots, stepping/tracing, compact and x64 scalar/SIMD cursor register/thread state, bounded guest memory reads with selectable TTD query policies, trace-backed memory range and buffer provenance queries, memory watchpoint replay with full TTD access masks and optional thread filters, and PEB-backed command-line extraction when `ttd_replay_bridge.dll` and TTD runtime DLLs are available. The CLI also has daemon-free `recipes` discovery for TimDbg-inspired diagnostic workflows, `context snapshot` for one-shot agent context capture with architecture/disassembly/nearest-symbol/timeline enrichment from a running daemon session, `remote` helpers that explain and generate DbgSrv versus NTSD/CDB command lines, `symbols diagnose`/`symbols inspect`/`symbols exports`/`symbols nearest` for symbol/binary/PDB/export readiness checks and nearest-export fallback, `source resolve` for trailing-component source path matching, `disasm`/`u` for x64 instruction analysis, `object vtable` for read-only COM/C++ object analysis, `stack recover` for corrupted-stack return-address candidates, and `memory dump`/`memory classify`/`memory strings`/`memory dps`/`memory chase` for string/fill/entropy/pointer/instruction hints, bounded string extraction, dps-style pointer rows, and bounded pointer-chain inspection.

## Build And Check Commands

Run these from the repository root:

```powershell
cargo fmt --check
cargo test --workspace
cargo clippy --workspace --all-targets
cargo build -p windbg-tool
```

The runnable debug server is:

```text
target/debug/windbg-tool.exe mcp
```

The same executable also supports:

```text
target/debug/windbg-tool.exe daemon start
target/debug/windbg-tool.exe daemon ensure
target/debug/windbg-tool.exe discover
target/debug/windbg-tool.exe recipes
target/debug/windbg-tool.exe context snapshot --session <id> --cursor <id>
target/debug/windbg-tool.exe remote explain
target/debug/windbg-tool.exe live capabilities
target/debug/windbg-tool.exe live launch --command-line "C:\Windows\System32\notepad.exe" --end detach
target/debug/windbg-tool.exe breakpoint capabilities
target/debug/windbg-tool.exe datamodel capabilities
target/debug/windbg-tool.exe target capabilities --session <id> --cursor <id>
target/debug/windbg-tool.exe symbols diagnose --session <id>
target/debug/windbg-tool.exe symbols inspect <exe-or-dll>
target/debug/windbg-tool.exe symbols exports <exe-or-dll> --filter <name>
target/debug/windbg-tool.exe symbols nearest --session <id> --cursor <id> --address <addr>
target/debug/windbg-tool.exe source resolve <recorded-path> --search-path <source-root>
target/debug/windbg-tool.exe module audit --session <id>
target/debug/windbg-tool.exe module search-order suspicious.dll --app-dir <app-dir>
target/debug/windbg-tool.exe architecture state --session <id> --cursor <id>
target/debug/windbg-tool.exe replay capabilities --session <id>
target/debug/windbg-tool.exe replay to --session <id> --cursor <id> --position 50

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awakecoding/windbg-tool](https://github.com/awakecoding/windbg-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
