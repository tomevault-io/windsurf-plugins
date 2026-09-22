---
trigger: always_on
description: Guidelines for AI agents working on this codebase.
---

# AGENTS.md - EdiabasX TypeScript

Guidelines for AI agents working on this codebase.

## Project Overview

TypeScript implementation of BMW EDIABAS (Electronic Diagnostic Basic System).
Migrating from C# EdiabasLib to a modern TypeScript monorepo.

---

## Repository Orientation

Read this first when resuming a session. The sections below cover the
workspace map, the cross-repo dependency, the release / deploy story,
and the limitations / gotchas accumulated in real use — so a new agent
doesn't have to derive them from git history + reading 20 package.jsons.

### Workspace layout

18 library packages under `packages/` + 2 end-user apps under `apps/`.
What each does:

| Path | Role |
|---|---|
| `packages/core` | CP1252 encoding, XOR (key `0xF7`) decryption, error codes, shared types, `IEdiabas` interface |
| `packages/best-parser` | PRG / GRP byte-level parser; disassembles the BEST2 bytecode |
| `packages/best-decompiler` | BEST/2 near-source decompiler — lifts bytecode to `.b2v` pseudo-source |
| `packages/interpreter` | BEST2 VM — registers, flags, call/value stacks, ~184 opcodes. The TS port of EdiabasLib's interpreter core |
| `packages/interface-base` | Abstract `EdiabasInterface` + in-memory `SimulationInterface` |
| `packages/interface-serial` | K-line / K+DCAN / serial transport, DS2 / KWP / ISO-TP / TP2.0 sessions. Browser-safe core; Node-only `/node` subpath ships the `serialport`-backed transport |
| `packages/interface-j2534` | SAE J2534 PassThru transport via Tactrix OpenPort 2.0. Frame-level integrity that K+DCAN UART bridges can't provide |
| `packages/interface-enet` | DoIP / HSFZ over Ethernet |
| `packages/interfaces` | Factory (`createInterface(name, opts)`), interface registry, JSON-RPC gateway server + client. Browser-safe `/client` subpath exports `GatewayClient` only |
| `packages/protocol-uds` | UDS (ISO 14229) service IDs, NRCs, ISO-TP framing |
| `packages/protocol-kwp` | KWP2000 / KWP1281 service IDs, NRCs |
| `packages/protocol-doip` | DoIP / HSFZ (ISO 13400) primitives — WIP |
| `packages/ediabas` | Main `Ediabas` class — loads PRG/GRP, drives the VM, returns grouped result sets. Exports `LOG_CATEGORIES` for UI/config surfaces |
| `packages/ediabasx-server` | JSON-RPC server — exposes `init`, `end`, `job`, `listSgbd`, `listJobs`, `getJobMetadata`, `disassembleJob`, `log.subscribe`/`log.unsubscribe` over TCP or WebSocket. Accepts external standard WebSockets via `attachStandardWebSocket()` (relay / Bimmerz Connect). Node-only |
| `packages/ediabasx-client` | JSON-RPC client (`EdiabasClient`) + embedded in-process wrapper (`EmbeddedEdiabas`), both implementing `IEdiabas`. Accepts pre-connected WebSockets via `socket` option (relay / Bimmerz Connect). Browser-safe `./client` subpath exports `EdiabasClient` only |
| `packages/host-config` | Shared loader for `~/.config/ediabasx/config.json` + interface-selection resolver + SGBD path resolution. Node-only (CJS) |
| `packages/mac-ftdi-latency` | macOS-only native addon: sets the FTDI USB-side latency timer via IOSSDATALAT ioctl |
| `packages/web-ui` | Shared Svelte 5 source-only components (`ConnectButton`, `ConnectConfigPanel`, `InterfaceConfigPanel`, `ModeConfigPanel`, `ServerConfigPanel`) + config types. Consumed by ediabasx-web, inpax-web, ncsx-web. Requires `@emdzej/bimmerz-theme` Tailwind preset |
| `apps/cli` | `ediabasx` terminal binary — `info` / `jobs` / `job` / `tables` / `table` / `decompile` / `run` / `explore` / `gateway` / `serve` (with `--connect` for Bimmerz Connect relay) / `simulator` / `configure` / `interfaces` / `docs` subcommands. Has a TUI for interactive job runs |
| `apps/web` | Browser SPA at `ediabasx.bimmerz.app` — two modes: **embedded** (local Web Serial / J2534 / gateway + File System Access, Chromium-only) and **client** (connects to a remote `ediabasx serve` instance over WebSocket, any browser). PWA-installable |

### Sibling repos (consumers)

`@emdzej/ediabasx-*` packages are consumed by three sibling monorepos
(single maintainer, same npm scope). Bumps here ripple through each
via npm pins (not workspace links):

| Repo | Location | What consumes ediabasx |
|---|---|---|
| **inpax** | `~/Projects/my/inpax` | `apps/cli`, `apps/inpax-web`, `packages/ediabasx-provider` |
| **ncsx** | `~/Projects/my/ncsx` | `apps/cli`, `apps/ncsx-web`, coding/NCS runtime |
| **nfsx** | `~/Projects/my/nfsx` | `apps/cli`, `nfsx-runtime`, flash/FSC orchestration |

When changing a public API surface, ping all consumers to update their
pins. Browser-bundling fixes propagate automatically via `^x.y` semver
ranges on next install.

### Server / client architecture

`packages/ediabasx-server` + `packages/ediabasx-client` form the
remote-diagnostics layer:

- **`EdiabasServer`** — JSON-RPC 2.0 server over TCP or WebSocket.
  Owns the cable, SGBD directory, and an `Ediabas` instance. Methods:
  `init`, `end`, `job`, `listSgbd`, `listJobs`, `getJobMetadata`,
  `disassembleJob`, `log.subscribe`, `log.unsubscribe`. Sends log
  entries as JSON-RPC notifications (`method: "log"`, no `id`).
  CLI: `ediabasx serve --sgbd-path <dir> --interface <name>`.
  `attachStandardWebSocket(ws)` accepts a pre-connected standard
  `WebSocket` (e.g. from Bimmerz Connect relay); `ensureBroadcastSink()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emdzej/ediabasx](https://github.com/emdzej/ediabasx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
