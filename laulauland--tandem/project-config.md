---
trigger: always_on
description: Execution guide for working on the `tandem` codebase.
---

# AGENTS

Execution guide for working on the `tandem` codebase.

## How to read these docs

1. Read `ARCHITECTURE.md` for system boundaries and project structure.
2. Read `docs/design-docs/workflow.md` for the orchestrator→agents→git workflow.
3. Read `docs/design-docs/jj-lib-integration.md` for trait signatures and registration.
4. Read `docs/exec-plans/tech-debt-tracker.md` for known issues to work on.
5. Check `docs/exec-plans/completed/` for context on how each slice was built.
6. Implement changes via failing integration test first.

## What tandem is

Tandem applies a **server-client model to jj's store layer**. The server hosts
a normal jj+git colocated repo. Agents on remote machines use the `tandem`
binary (which embeds jj-cli with a custom tandem backend) to read and write
objects over Cap'n Proto RPC.

The server is the **point of origin** — it typically runs on a VM/VPS as a
long-running service. It's where git operations happen (`jj git push`,
`jj git fetch`, `gh pr create`). The orchestrator/teamlead runs these on the
server to ship code upstream. The tandem server is the source of truth, with
GitHub as a mirror.

## Installation

Published on [crates.io](https://crates.io/crates/jj-tandem) as `jj-tandem`:

```bash
cargo install jj-tandem
```

Requires a Rust toolchain. No system `capnp` binary is required for normal installs/builds.
Or build from source: `cargo build --release`.

`build.rs` compiles schema bindings when `capnp` is available and falls back to
checked-in `src/tandem_capnp.rs` when it is not.

Maintainers only: when changing `schema/tandem.capnp`, regenerate checked-in bindings:
`TANDEM_REGENERATE_BINDINGS=1 cargo build`.

## Single binary, three modes

```
tandem up --repo <path> --listen <addr>       # start background daemon
tandem serve --listen <addr> --repo <path>    # foreground server (systemd/docker)
tandem [jj args...]                           # client mode (stock jj via CliRunner)
```

Plus lifecycle commands that talk to a running server:

```
tandem down                                   # stop daemon
tandem server status [--json]                 # health check
tandem server logs [--level <level>] [--json] # stream logs from daemon
```

The client mode is `CliRunner::init().add_store_factories(tandem_factories()).run()`.
All stock jj commands work transparently: `tandem status`, `tandem new`,
`tandem log`, `tandem diff`, `tandem file show`, `tandem bookmark create`
are all jj commands running through our binary.

Server mode embeds jj-lib and uses the Git backend internally. When a client
calls `putObject(file, bytes)`, the server stores the object. Objects are real
jj-compatible blobs — `jj git push` on the server just works.

`tandem up` is the easy way to start the server — it forks `tandem serve --daemon`
in the background, waits for the control socket to become healthy, prints the PID,
and exits. `tandem serve` is the foreground mode for systemd, Docker, or debugging.
Both create a control socket so `tandem down`, `tandem server status`, and
`tandem server logs` work against either.

## Source layout

```
src/
  main.rs              CLI dispatch (clap) + CliRunner passthrough
  tandem_capnp.rs      Generated Cap'n Proto bindings (checked in)
  server.rs            Server — jj Git backend + Cap'n Proto RPC
  control.rs           Control socket — daemon management (Unix socket, JSON lines)
  backend.rs           TandemBackend (jj-lib Backend trait)
  op_store.rs          TandemOpStore (jj-lib OpStore trait)
  op_heads_store.rs    TandemOpHeadsStore (jj-lib OpHeadsStore trait)
  rpc.rs               Cap'n Proto RPC client wrapper
  proto_convert.rs     jj protobuf ↔ Rust struct conversion
  watch.rs             tandem watch command
schema/
  tandem.capnp         Cap'n Proto schema (Store + HeadWatcher)
build.rs               Build-time schema generation with checked-in fallback
tests/
  common/mod.rs        Test harness (server spawn, HOME isolation)
  slice1-7 tests       Core integration tests (file round-trip, visibility, CAS, git)
  slice10-13 tests     Server lifecycle tests (shutdown, control socket, up/down, logs)
```

## Docs layout

```
docs/
  README.md                          Overview and pointers
  design-docs/
    workflow.md                      Orchestrator→agents→git workflow
    jj-lib-integration.md            Trait signatures and store registration
    rpc-protocol.md                  Cap'n Proto protocol details
    rpc-error-model.md               Error handling conventions
    server-lifecycle.md              tandem up/down + server status/logs design
    core-beliefs.md                  Design principles
  exec-plans/
    completed/                       Completion notes for all 13 slices
    tech-debt-tracker.md             Known issues (P1/P2/P3)
  product-specs/
    core-product.md                  Product intent and scope
```

## Critical invariants

1. **The client is stock `jj`.** Tandem implements jj-lib's `Backend`, `OpStore`,
   and `OpHeadsStore` traits as Cap'n Proto RPC stubs. There is no custom
   `tandem new/log/describe/diff` CLI — those are all jj commands.

2. **Tests assert on file bytes, not descriptions.** Every integration test
   must verify file content round-trips correctly via `jj cat`. Description-only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laulauland/tandem](https://github.com/laulauland/tandem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
