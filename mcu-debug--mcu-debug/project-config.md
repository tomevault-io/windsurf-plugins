---
trigger: always_on
description: This file captures architectural facts that are not obvious from reading the code alone. Read this before making changes to the debug adapter, proxy, or RTT subsystems.
---

# MCU Debug — AI Agent Context

This file captures architectural facts that are not obvious from reading the code alone. Read this before making changes to the debug adapter, proxy, or RTT subsystems.

---

## Key Reference Documents

| Document                                                       | What it covers                                                                                                    |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| [docs-internal/Proxy-Plan.md](docs-internal/Proxy-Plan.md)     | Definitive topology for remote probe support — two scenarios, terminology, Funnel Protocol                        |
| [docs-internal/ARCHITECTURE.md](docs-internal/ARCHITECTURE.md) | High-level architecture; may have drifted from current implementation in details, but the overall arch is correct |
| [docs/rtt.md](docs/rtt.md)                                     | RTT implementation — this project's approach is a superset of the standard gdb-server model                       |

---

## Critical: Terminology Inversion

**VS Code's "Local" / "Remote" terminology is inverted relative to the intuitive meaning in this project.**

- VS Code calls the machine with the USB probe **"Local"** (the UI side).
- VS Code calls the machine running the extension's workspace (WSL / container / SSH) **"Remote"**.
- The `mcu-debug` UI extension runs on the **VS Code Local** side (probe host).
- The Debug Adapter (DA) and GDB run on the **VS Code Remote** side (workspace / engineer's source).

When the Proxy-Plan.md says "Engineer Machine" and "Probe Host", use those terms — they are unambiguous. Do not use "local" or "remote" without qualifying which convention you mean.

---

## Debug Adapter is Three Components, Not One

The debug adapter is **not** a single TypeScript process. It has three cooperating parts:

1. **TypeScript DA** (`packages/mcu-debug/src/adapter/`) — The DAP server. Talks to VS Code, manages sessions, orchestrates GDB via stdio.

2. **`da_helper`** (Rust, `packages/mdbg/src/da_helper/`) — A Rust binary invoked by the TS DA as a subprocess. Responsible for ELF parsing, symbol table lookup, and disassembly (via objdump + Capstone). The TS side does **not** parse ELF directly. Any feature that requires symbol information goes through this helper.

3. **Proxy client** (`packages/mdbg/src/proxy_helper/`) — Also Rust. Implements the client side of the Funnel Protocol for reaching a Probe Agent on a remote/host machine. Used when the probe is not accessible directly from the DA process (WSL, Dev Container, or LAB topology).

The `mdbg` binary is a single Rust binary with subcommands (`da-helper`, `proxy`, …). Do not assume these are separate binaries.

---

## Remote Probe Topologies

There are two distinct scenarios. See [Proxy-Plan.md](docs-internal/Proxy-Plan.md) for full detail.

**Topology A — VS Code Remote (WSL / Dev Container)**
- DA runs inside WSL or a container; probe is on the host machine.
- The `mcu-debug` **UI extension** (not the DA) runs on the host and spawns the Probe Agent.
- DA reaches the Probe Agent via `127.0.0.1` (WSL mirrored mode) or `host.docker.internal`.
- This is the `type: "auto"` case in config.

**Topology B — LAB (physically separate machine)**
- DA and all tooling run on the engineer's machine; probe is on a lab server.
- An SSH tunnel (`ssh -L`) is established by the UI extension.
- The DA sees "ghost ports" on `127.0.0.1` that tunnel through to the lab server's Probe Agent.
- No inbound firewall rules are needed on the lab server.

**Probe Agent** (`mdbg proxy`) always runs on the machine physically attached to the probe. It manages gdb-server lifecycle and implements the Funnel Protocol.

---

## Module Boundary Rules (`packages/mcu-debug/src/`)

These rules are **hard constraints**. Enforce them on every change.

### 1. VS Code APIs are confined to `frontend/`

Only files inside `src/frontend/` may import from `vscode` or use any `vscode.*` API.
`common/`, `adapter/`, and `cli/` must never import `vscode`.

### 2. Nothing outside `frontend/` may import from `frontend/`

`common/`, `adapter/`, and `cli/` must never import a file whose path contains `src/frontend/`.
`frontend/` is a consumer of `common/` — not a library for it.

### 3. Platform differences go through `IHostAdapter`

When behaviour differs between the VS Code extension and the CLI, the difference is expressed through the `IHostAdapter` interface (`common/host-adapter.ts`).

- **`VscodeAdapter`** (`frontend/vscode-adapter.ts`) — calls `vscode.*` APIs.
- **`CliAdapter`** (`cli/cli-adapter.ts`, to be created) — writes to the mux stream / logger.

`adapter/` (the DAP server) does **not** use `IHostAdapter`. It conforms to the DAP protocol and has no platform-specific UI calls.

### 4. Logging via `logger`, not `console` or `MCUDebugChannel`

Use `logger` from `common/logger.ts` in `cli/` only. Use getHostAdapter().debugMessage in `common/` and `frontend/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcu-debug/mcu-debug](https://github.com/mcu-debug/mcu-debug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
