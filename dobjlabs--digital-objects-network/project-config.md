---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository is the reference implementation of the Digital Objects Network: a decentralized network for creating, executing, and exchanging Digital Objects -- fully programmable state machines owned and operated by Internet users, that can be passed between mutually untrusting users while keeping their integrity and consistency, without relying on any central trusted authority. Objects are privately held files on disk; their state transitions are proved with POD2/Plonky2 and anchored to Ethereum blob data availability, so the chain sees only opaque commitments.

The repo ships a headless daemon (`dobjd`) that owns all driver state, several clients that drive it over HTTP/SSE/MCP (a React GUI servable in a browser or wrapped in a Tauri shell, the `dobj` CLI, an MCP server for AI agents), and the chain-side services that anchor and sync objects. The bundled `craft-basics` plugin (a small crafting game) is the demonstrated end-to-end flow.

This file focuses on navigating the code, building/testing, and gotchas.

## Workspace layout

The workspace is declared in `Cargo.toml`. Crate-by-crate:

| Crate                            | Role                                                                                                                                                                                                 |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dobjd`                          | **The daemon.** Headless HTTP server on `:7717` wrapping the driver. Every client talks to it.                                                                                                       |
| `cli`                            | `dobj` CLI binary. Thin HTTP/SSE client of dobjd. No `Driver` of its own.                                                                                                                            |
| `interfaces/gui/src-tauri`       | Tauri 2 shell. Holds **no** driver state — webview talks to dobjd over HTTP. Native conveniences only.                                                                                               |
| `interfaces/gui/src` (TS)        | React/Vite frontend. Component-based: `features/{actions,objects,context,proof-runner,settings}`.                                                                                                    |
| `driver`                         | Headless Rust orchestration library. **The core.** Owns `~/.dobj/`, runs actions end-to-end.                                                                                                         |
| `sdk`                            | Rhai engine + two-phase Loader/Executor that compiles plugin scripts into pod2 modules.                                                                                                              |
| `txlib`                          | Transaction state machine: `StateHeader`, `GroundingWitness`, `Tx`, `TxBuilder` + `TxFinalized` rule.                                                                                                |
| `synchronizer`                   | Long-running service: ingests Ethereum blobs, maintains Merkle state, serves HTTP queries.                                                                                                           |
| `relayer`                        | HTTP service that wraps proofs as EIP-4844 blob txs and submits them.                                                                                                                                |
| `archiver`                       | Service that follows beacon blocks and archives blobs filtered by destination address to the filesystem (no DB). Serves them via a beacon-compatible HTTP API; the synchronizer reads blobs from it. |
| `eth-clients`                    | Partial Ethereum Beacon client API (adapted from Blobscan, MIT). Used by `archiver`/`synchronizer` to follow the chain.                                                                              |
| `payload`                        | Cross-crate types: blob payload encoding, plonky2 proof shrink wrapper, `BlobParser`.                                                                                                                |
| `wire-types`                     | Pure-data types crossing process boundaries (HTTP/MCP/SSE/CLI). Dependency-light — no pod2/plonky2.                                                                                                  |
| `pod2utils`                      | Macros (`st_custom!`, `dict!`, `set!`, `op!`, …) and `BuildContext` for loading podlang modules.                                                                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dobjlabs/digital-objects-network](https://github.com/dobjlabs/digital-objects-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
