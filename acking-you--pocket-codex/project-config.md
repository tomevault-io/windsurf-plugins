---
trigger: always_on
description: > This document is the contract between human contributors, AI coding
---

# Pocket-Codex Agent Guide

> This document is the contract between human contributors, AI coding
> agents (Claude Code, Codex CLI, etc.) and the project itself. Read it
> before touching code.

## 1. Project intent

Pocket-Codex turns the upstream
[`codex app-server`](https://github.com/openai/codex) protocol into a
portable, multi-device experience, and additionally exposes the host's
Codex login as a relay-reachable Responses API endpoint for any device:

- A **pure-Rust CLI** (`pocket-codex`) supervises a local
  `codex app-server` process on the machine that already has Codex
  installed.
- The same CLI ships an in-process **Responses API proxy** that reuses
  the host's `codex login` (ChatGPT account or `CODEX_ACCESS_TOKEN`)
  to serve OpenAI-compatible `/v1/responses` HTTP + WebSocket traffic,
  letting devices *without* Codex installed drive the same model
  through the relay.
- The CLI uses [`pb-mapper`](https://github.com/acking-you/pb-mapper)
  to **register** either service on a relay under
  `pcx:<device>:<kind>:<name>` keys, or to **subscribe** to remote
  ones, materialising them as local TCP endpoints.
- A **Flutter front-end** (under `apps/flutter`, driven through
  `flutter_rust_bridge`) consumes the app-server JSON-RPC protocol
  directly to give every platform a native UI without re-implementing
  the model runtime.

Two ways to wire devices together, both first-class:

- **Self-host** — every device shares one relay address plus a 32-byte
  `MSG_HEADER_KEY`, and talks to the relay directly under `pcx:…` keys.
  Selected by an explicit `--relay`.
- **Hosted account** — the optional `pocket-codex-backend` runs once on a
  server; devices sign in with GitHub, and the backend hands each account a
  short-lived relay credential confined to its own `pcxu:<user>:…`
  namespace. Devices then talk to the relay **directly**: the relay's
  administrator key never reaches a client, accounts stay isolated from
  each other, and the backend is not on the data path.

The repository deliberately does **not** vendor a model runtime; the
user-supplied `codex` binary (and its login state) is the source of
truth.

## 2. Repository layout

```
apps/flutter/              # Flutter UI (FRB-driven, FVM-locked at 3.44.0)
assets/logo/               # Project artwork (poster.png, logo.png)
crates/                    # all first-party Rust crates; see §3 for who owns what
deploy/                    # hosted-backend deployment unit + config examples
deps/
  codex/                   # acking-you/codex fork, branch `pocket-codex`
                           # (git submodule) = upstream openai/codex main +
                           # our adaptations; see §8
docs/                      # design notes, protocol references, CLI verification
scripts/                   # install scripts, local CI, CI affected-surface gate
```

`Cargo.toml` is a workspace root; every crate under `crates/` is a
workspace member (see the `members` list for the canonical set).
The Codex submodule under `deps/` is kept **out** of the workspace via the
`exclude` list and retains its upstream lints/profiles. Cargo fetches pb-mapper
from its dedicated `pocket-codex` Git branch; `Cargo.lock` pins the exact commit
and its registry dependencies. No pb-mapper, kanal, or uni-stream submodules
or local dependency patches are needed.

## 3. Crate responsibilities

Shared / host side:

| Crate                       | Owns                                                                                           |
| --------------------------- | ---------------------------------------------------------------------------------------------- |
| `pocket-codex-core`         | configuration schema, on-disk `state.toml`, well-known paths, error types, `service::{ServiceId, ServiceKind, sanitize_component, default_device_id}` for `pcx:<device>:<kind>:<name>` relay keys — small, dependency-light |
| `pocket-codex-codex`        | spawning / supervising / inspecting the external `codex app-server` child process, upstream protocol types and compatible JSON-RPC envelopes |
| `pocket-codex-pb`           | async wrappers around the Git-pinned `pb-mapper` client SDK: `RelaySession` (address + credential), register / subscribe / status, `publish` (and the one name-conflict failure a caller must not retry), admin credential issuance, and credential keep-alive |
| `pocket-codex-api-proxy`    | local Responses API proxy: forwards `/v1/responses` (HTTP + WS) to ChatGPT's Codex backend, reusing the host's `codex login`; shared by the CLI worker and the in-app host |
| `pocket-codex-host-svc`     | host-side meta service — remote-viewable codex sessions, per-thread config, attachment upload — published on the relay as a third `meta:<name>` service |
| `pocket-codex-cli`          | user-facing `pocket-codex` binary; account (`login` / `logout` / `account`), setup (`init`), high-level `serve` / `connect` / `api {serve,connect}` / `services {list,default set}` / `status` / `stop`, low-level `codex {start,stop,status}`, `pb {register,subscribe,status}`, `remote-hint`, `version` |
| `pocket_codex_bridge`       | `cdylib + staticlib` consumed by Flutter via `flutter_rust_bridge`; auto-generated bindings live in `lib/src/rust` of the Flutter app |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acking-you/pocket-codex](https://github.com/acking-you/pocket-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
